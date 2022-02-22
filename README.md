# Setup Next.js

- https://nextjs.org/docs

# Setup Tailwindcss

- https://tailwindcss.com/docs/guides/nextjs

# Installing storybook & addons

- `npx sb init`
- `yarn add -D @storybook/builder-webpack5`
- `yarn add -D @storybook/manager-webpack5`
- `yarn add -D @storybook/addon-postcss`

## Add Webpack as a resolution dependency

```
// package.json

"resolutions": {
    "webpack": "^5"
}
```

## Replace `.storybook/main.js`

```
// .storybook/main.js

const path = require('path');

module.exports = {
  stories: ['../**/*.stories.mdx', '../**/*.stories.@(js|jsx|ts|tsx)'],
  /** Expose public folder to storybook as static */
  staticDirs: ['../public'],
  addons: [
    '@storybook/addon-links',
    '@storybook/addon-essentials',
    {
      /**
       * Fix Storybook issue with PostCSS@8
       * @see https://github.com/storybookjs/storybook/issues/12668#issuecomment-773958085
       */
      name: '@storybook/addon-postcss',
      options: {
        postcssLoaderOptions: {
          implementation: require('postcss'),
        },
      },
    },
  ],
  core: {
    builder: 'webpack5',
  },
  webpackFinal: (config) => {
    /**
     * Add support for alias-imports
     * @see https://github.com/storybookjs/storybook/issues/11989#issuecomment-715524391
     */
    config.resolve.alias = {
      ...config.resolve?.alias,
      '@': [path.resolve(__dirname, '../'), path.resolve(__dirname, '../')],
    };

    /**
     * Fixes font import with /
     * @see https://github.com/storybookjs/storybook/issues/12844#issuecomment-867544160
     */
    config.resolve.roots = [
      path.resolve(__dirname, '../public'),
      'node_modules',
    ];

    return config;
  },
};
```

## Replace `.storybook/preview.js`

```
// .storybook/preview.js

import '../styles/globals.css';
import * as NextImage from 'next/image';

const OriginalNextImage = NextImage.default;

Object.defineProperty(NextImage, 'default', {
  configurable: true,
  value: (props) => <OriginalNextImage {...props} unoptimized />,
});

export const parameters = {
  actions: { argTypesRegex: '^on[A-Z].*' },
  controls: {
    matchers: {
      color: /(background|color)$/i,
      date: /Date$/,
    },
  },
  previewTabs: {
    'storybook/docs/panel': { index: -1 },
  },
  viewMode: 'docs',
  docs: {
    source: {
      state: 'open',
     },
  },
};
```

## Example

Check `./stories/Button.jsx` line 13

### Note

Open source code by default

```
  docs: {
    source: {
      state: 'open',
     },
  },
```

# Run

- `yarn storybook`

# Reference

- https://theodorusclarence.com/blog/nextjs-storybook-tailwind
