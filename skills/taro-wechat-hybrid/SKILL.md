---
name: taro-wechat-hybrid
description: Guide for Taro + WeChat Mini Program hybrid development, covering architecture patterns, component integration, routing, state management, and performance optimization when mixing Taro React components with native WeChat Mini Program components and APIs.
---

# Taro + WeChat Mini Program Hybrid Development

This skill provides guidance for developing applications that combine Taro (React-based framework) with native WeChat Mini Program components and APIs. It covers architecture patterns, integration strategies, and best practices for maintaining code quality and performance in hybrid environments.

## Overview

Hybrid development allows you to leverage Taro's cross-platform capabilities while utilizing native WeChat Mini Program features for optimal performance and platform-specific functionality. This approach enables gradual migration, component reuse, and access to native APIs that may not be fully abstracted by Taro.

## Architecture Patterns

### Project Structure

```
project/
├── src/                    # Taro source code
│   ├── pages/             # Taro pages (React components)
│   ├── components/        # Taro components
│   ├── components/        # Native components (.wxml, .wxss, .js, .json)
│   ├── wxs/              # WXS scripts for performance-critical logic
│   ├── app.tsx            # Taro app entry
│   └── app.config.ts      # App configuration
├── dist/                  # Compiled output (WeChat Mini Program format)
└── config/                # Taro build configuration
```

### Component Registration

Native components are registered in `app.config.ts` using `usingComponents`:

```typescript
export default defineAppConfig({
  usingComponents: {
    'custom-wrapper': './components/custom-wrapper',
    'nav-bar': './components/nav-bar',
    'ui-button': './library/ui-button',
  }
});
```

**Key Points:**
- Component names must start with lowercase letters
- Paths are relative to the `src` directory
- Native components can be used in both Taro and native pages
- Taro components can be used in native pages via registration

### Build Configuration

Taro configuration (`config/index.ts`) handles compilation:

```typescript
const config = {
  sourceRoot: 'src',
  outputRoot: 'dist',
  framework: 'react',
  compiler: {
    type: 'webpack5',
  },
  copy: {
    patterns: [
      { from: 'src/static/', to: 'dist/static/' },
      { from: 'src/wxs/', to: 'dist/wxs/' },
    ],
  },
  mini: {
    postcss: {
      cssModules: {
        enable: true,
        config: {
          namingPattern: 'module',
          generateScopedName: '[name]__[local]___[hash:base64:5]',
        },
      },
    },
  },
};
```

## Integration Strategies

### Using Native APIs

Direct access to WeChat Mini Program APIs is available:

```typescript
// Storage operations
const storageInfo = wx.getStorageInfoSync();
const userInfo = wx.getStorageSync('userInfo');
wx.setStorageSync('token', token);

// Navigation
wx.navigateTo({ url: '/pages/target/index' });
wx.redirectTo({ url: '/pages/target/index' });

// System info
const systemInfo = wx.getSystemInfoSync();

// Location
const location = await wx.getLocation({ type: 'wgs84' });
```

**Best Practices:**
- Use `wx.` APIs directly when Taro abstractions don't meet requirements
- Prefer Taro APIs (`Taro.navigateTo`, `Taro.getStorage`) for cross-platform compatibility
- Wrap native API calls in try-catch for error handling
- Consider creating wrapper utilities for common native operations

### Routing and Navigation

#### Proxy Pattern for Route Enhancement

Intercept and enhance navigation calls:

```typescript
const initProxyRouterEvent = (): void => {
  const _navigateTo = Taro.navigateTo;
  
  Taro.navigateTo = (option) => {
    let { url } = option;
    // Add query parameters for specific routes
    if (shouldAddParams(url)) {
      url = appendParams(url, { source: 'fromApp' });
    }
    return _navigateTo({ ...option, url });
  };
  
  // Expose native methods
  wx.navigateToInsideMode = Taro.navigateTo;
};
```

**Considerations:**
- Maintain consistent routing behavior across Taro and native pages
- Handle query parameters and deep linking
- Support both internal and external navigation modes
- Test navigation flows thoroughly in hybrid scenarios

### Native Component Structure

Native components follow standard WeChat Mini Program structure:

```
component/
├── index.wxml      # Template
├── index.wxss      # Styles
├── index.js        # Logic (can use ES6+)
└── index.json      # Component config
```

**Example Native Component:**

```javascript
// index.js
import { connect } from '@/behaviors/store';

const storeBehavior = connect((state) => ({
  user: state.user,
  app: state.app,
}));

Component({
  behaviors: [storeBehavior],
  properties: {
    loading: { type: Boolean, value: false },
    customNavigation: { type: Boolean, value: false },
  },
  lifetimes: {
    attached() {
      const pages = getCurrentPages();
      const currentPage = pages[pages.length - 1];
      // Initialize component
    },
    ready() {
      // Component ready
    },
  },
  pageLifetimes: {
    show() {
      // Page show lifecycle
    },
  },
  methods: {
    handleEvent(e) {
      this.triggerEvent('customEvent', e.detail);
    },
  },
});
```

### WXS for Performance

Use WXS (WeiXin Script) for performance-critical template logic:

```javascript
// wxs/utils.wxs
function formatPrice(number) {
  return number.toLocaleString();
}

function validateType(checkType, type, subType) {
  // Validation logic based on your business rules
  return checkType === type;
}

module.exports = {
  formatPrice: formatPrice,
  validateType: validateType,
};
```

**Usage in Templates:**

```xml
<wxs src="../../wxs/utils.wxs" module="utils" />
<text>{{utils.formatPrice(price)}}</text>
```

**When to Use WXS:**
- Template-level data formatting
- Simple validation logic
- Performance-critical rendering operations
- Avoid complex business logic in WXS

## State Management

### Redux Integration

Use Redux for shared state across Taro and native components:

```typescript
// Store setup
import { Provider } from 'react-redux';
import configStore from './store';

const store = configStore();

// In Taro app
class App extends Component {
  render() {
    return <Provider store={store}>{this.props.children}</Provider>;
  }
}

// In native components
import { connect } from '@/behaviors/store';

const storeBehavior = connect((state) => ({
  user: state.user,
  app: state.app,
}));

Component({
  behaviors: [storeBehavior],
  // Access via this.data.user, this.data.app
});
```

**Patterns:**
- Centralized store for shared application state
- Behavior-based connection for native components
- Provider pattern for Taro components
- Avoid duplicating state initialization

## Lifecycle Management

### App Lifecycle

```typescript
class App extends Component {
  async componentDidMount() {
    // Equivalent to onLaunch
    // Initialize SDKs, check network, setup proxies
  }
  
  componentDidShow() {
    // Equivalent to onShow
    // Handle app visibility, refresh data
  }
  
  componentDidHide() {
    // Equivalent to onHide
    // Cleanup, pause operations
  }
}
```

### Page Lifecycle

Taro pages use React lifecycle methods:
- `componentWillMount` / `componentDidMount` → `onLoad`
- `componentDidShow` → `onShow`
- `componentDidHide` → `onHide`
- `componentWillUnmount` → `onUnload`

Native pages use standard WeChat Mini Program lifecycle:
- `onLoad`, `onShow`, `onReady`, `onHide`, `onUnload`

**Coordination:**
- Ensure initialization order is correct
- Handle state synchronization between Taro and native pages
- Clean up subscriptions and timers appropriately

## Styling Strategies

### CSS Modules

Enable CSS Modules for style isolation:

```typescript
// config/index.ts
mini: {
  postcss: {
    cssModules: {
      enable: true,
      config: {
        namingPattern: 'module',
        generateScopedName: '[name]__[local]___[hash:base64:5]',
      },
    },
  },
}
```

### Shared Styles

Use global styles for common patterns:

```scss
// src/static/styles/_index.scss
@import './variables';
@import './mixins';
@import './base';
```

Register in config:

```typescript
sass: {
  resource: [path.resolve(__dirname, '..', 'src/static/styles/_index.scss')],
}
```

**Guidelines:**
- Use CSS Modules for component-specific styles
- Global styles for shared design tokens
- Maintain consistent spacing and typography
- Test style conflicts between Taro and native components

## Subpackage Configuration

Optimize bundle size with subpackages:

```typescript
// app.config.ts
export default defineAppConfig({
  subPackages: [
    {
      root: 'pages/feature-a/',
      name: 'feature-a',
      pages: ['detail', 'list'],
    },
    {
      root: 'pages/feature-b/',
      name: 'feature-b',
      pages: ['pages/profile/index', 'pages/settings/index'],
    },
  ],
  preloadRule: {
    'pages/feature-b/pages/profile/index': {
      packages: ['__APP__'],
    },
  },
});
```

**Optimization Tips:**
- Place less frequently used pages in subpackages
- Use preload rules strategically
- Monitor main package size
- Consider component-level code splitting

## Performance Optimization

### Code Splitting

```typescript
// config/index.ts
mini: {
  commonChunks(commonChunks) {
    // Let webpack automatically determine common chunks
    return commonChunks;
  },
}
```

### Image Optimization

- Use appropriate image formats (WebP when supported)
- Implement lazy loading for images
- Optimize image sizes before inclusion
- Use CDN for static assets

### Bundle Analysis

Monitor bundle size and composition:

```typescript
// Use webpack-bundle-analyzer in development
import { BundleAnalyzerPlugin } from 'webpack-bundle-analyzer';
```

## Common Patterns

### Error Handling

```typescript
try {
  const result = await wx.login();
  // Handle success
} catch (error) {
  if (error.errMsg.includes('频繁')) {
    // Handle rate limiting
  } else {
    // Handle other errors
  }
}
```

### Storage Management

```typescript
// Centralized storage utilities
const AUTH_STORAGE_KEYS = ['token', 'userInfo', 'sessionId'];

function clearAuth() {
  AUTH_STORAGE_KEYS.forEach((key) => {
    if (wx.getStorageInfoSync().keys.includes(key)) {
      wx.removeStorageSync(key);
    }
  });
}
```

### System Information

```typescript
function getSystemInfo() {
  const systemInfo = wx.getSystemInfoSync();
  return {
    statusBarHeight: systemInfo.statusBarHeight,
    windowHeight: systemInfo.windowHeight,
    windowWidth: systemInfo.windowWidth,
  };
}
```

## Testing Considerations

### Development Workflow

1. **Development**: `npm run dev:weapp` - Watch mode with hot reload
2. **Build**: `npm run build:weapp` - Production build
3. **Preview**: Open `dist/` in WeChat Developer Tools

### Debugging

- Use WeChat Developer Tools console
- Check network requests in developer tools
- Monitor performance with performance panel
- Test on real devices for accurate behavior

## Best Practices

1. **Component Selection**
   - Use Taro components for cross-platform compatibility
   - Use native components for platform-specific features
   - Document component usage patterns

2. **API Usage**
   - Prefer Taro APIs for cross-platform code
   - Use native APIs when Taro doesn't support needed features
   - Create abstraction layers for complex native operations

3. **State Management**
   - Centralize shared state in Redux
   - Use local state for component-specific data
   - Avoid prop drilling across hybrid boundaries

4. **Performance**
   - Monitor bundle size regularly
   - Use subpackages for large features
   - Optimize images and assets
   - Profile performance on real devices

5. **Code Organization**
   - Keep Taro and native code clearly separated
   - Use consistent naming conventions
   - Document hybrid integration points
   - Maintain clear component boundaries

## Common Pitfalls

1. **Lifecycle Mismatches**: Ensure lifecycle methods are called in correct order
2. **State Synchronization**: Watch for stale state between Taro and native components
3. **Style Conflicts**: CSS Modules help but test thoroughly
4. **Bundle Size**: Monitor and optimize regularly
5. **API Compatibility**: Check WeChat Mini Program base library version requirements
6. **Navigation Issues**: Test all navigation paths, especially deep links
7. **Storage Limits**: Be aware of storage quotas and cleanup strategies

## Examples

### Example 1: Using Native Component in Taro Page

```tsx
// Taro page
import { View } from '@tarojs/components';

export default function MyPage() {
  return (
    <View>
      <custom-wrapper loading={false} showNav={true}>
        {/* Page content */}
      </custom-wrapper>
    </View>
  );
}
```

### Example 2: Native Component Using Taro Store

```javascript
// Native component
import { connect } from '@/behaviors/store';

const storeBehavior = connect((state) => ({
  user: state.user,
}));

Component({
  behaviors: [storeBehavior],
  methods: {
    checkUserStatus() {
      const { user } = this.data;
      if (user.isLoggedIn) {
        // Handle logged in state
      }
    },
  },
});
```

### Example 3: Route Proxy Enhancement

```typescript
// Add tracking parameters to specific routes
const routeConfigs = [
  {
    add: [
      ['source', 'app'],
      ['mode', 'internal'],
    ],
    whiteList: ['/pages/product/detail', '/pages/order/detail'],
  },
];
```

## Resources

- [Taro Documentation](https://docs.taro.zone/en/docs/)
- [WeChat Mini Program Documentation](https://developers.weixin.qq.com/miniprogram/dev/framework/)
- Taro version: 3.6.34
- Framework: React 18.x

## Guidelines

- Always test hybrid integrations on real devices
- Monitor bundle size and performance metrics
- Document component usage and integration patterns
- Maintain clear separation between Taro and native code
- Use TypeScript for better type safety
- Follow WeChat Mini Program best practices for native components
- Keep dependencies up to date and compatible
