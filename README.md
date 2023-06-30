# Next-Iubenda

<span class="badge-lifecycle"><a href="https://github.com/mep-agency#lifecycle-policy" title="Check out our lifecycle stages"><img src="https://img.shields.io/badge/lifecycle-experimental-orange" alt="Project lifecyfle stage" /></a></span>
<span class="badge-license"><a href="https://github.com/mep-agency/next-iubenda" title="View this project on GitHub"><img src="https://img.shields.io/github/license/mep-agency/next-iubenda" alt="Project license" /></a></span>
<span class="badge-npmversion"><a href="https://www.npmjs.com/package/@mep-agency/next-iubenda" title="View this project on NPM"><img src="https://img.shields.io/npm/v/%40mep-agency/next-iubenda" alt="NPM version" /></a></span>
<span class="badge-npmdownloads"><a href="https://www.npmjs.com/package/@mep-agency/next-iubenda" title="View this project on NPM"><img src="https://img.shields.io/npm/dt/%40mep-agency/next-iubenda" alt="NPM downloads" /></a></span>

Do you use the [Iubenda](https://www.iubenda.com/)'s Cookie Solution, but have trouble integrating it properly into your Next.js projects? Then this is the tool for you.

Our goal is to create a simple, yet powerful tool to do the following:

- Display Iubenda's cookie banner, with the ability to customize every single configuration option
- Make sure we can prevent code execution of components that may require specific consent if it's not granted, as well as have a fallback widget that provides users with a call to action to change their preferences
- Take advantage of support for server-side and client-side components to improve performance
- Provide developers with a good DX thanks to features like
  - Type safety for configuration options
  - Templates for basic needs and hooks for advanced use cases
  - reasonable styling defaults, but full customization options

This package gives you these features with minimal effort and an extremely small footprint on your codebase.

# Missing features

- **Support to regulations other than the European GDPR**  
  Iubenda does a great job of supporting many regulations and helping developers make sure their websites/apps can be compliant worldwide. But this tool is currently limited to the European GDPR because that's what we know best. **Any help in adding support for additional features is appreciated.**
- **Shipping the package as pre-compiled**  
  Currently, this package is shipped as plain TypeScript files. While this doesn't seem to be a problem and may allow Next.js to better optimize the code, it requires the "transpilePackages" option to be enabled for this package, which may not be ideal. **Any suggestions/contributions in this regard are appreciated.**

# Project lifecycle, contribution and support policy

Our policies are available on our [main oranization page](https://github.com/mep-agency#projects-lifecycle-contribution-and-support-policy).

## Original authors

- Marco Lipparini ([liarco](https://github.com/liarco))
- Ivan Balmita ([ivanbalmita](https://github.com/ivanbalmita))

## Getting started

Simply install the package using any package manager:

```bash
# With Yarn
$ yarn add --dev @mep-agency/next-iubenda

# With NPM
$ npm install --save-dev @mep-agency/next-iubenda
```

At the moment this package is distributed as plain TypeScript code so you have to make sure Next.js will transpile it:

```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  transpilePackages: ['@mep-agency/next-iubenda'],
};

module.exports = nextConfig;
```

Update your main layout file to wrapp any part of the page which will contain consent-aware components.

Here is an example with the App Router:

```tsx
// ./app/layout.tsx

import './globals.css';
import { Inter } from 'next/font/google';
import { IubendaProvider, IubendaCookieSolutionBannerConfigInterface, i18nDictionaries } from '@mep-agency/next-iubenda';

const inter = Inter({ subsets: ['latin'] });

export const metadata = {
  title: 'Create Next App',
  description: 'Generated by create next app',
};

const iubendaBannerConfig: IubendaCookieSolutionBannerConfigInterface = {
  siteId: 0000000, // Your site ID
  cookiePolicyId: 00000000, // Your cookie policy ID
  lang: 'en',

  // See https://www.iubenda.com/en/help/1205-how-to-configure-your-cookie-solution-advanced-guide
};

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <>
      <html lang="en">
        <body className={inter.className}>
          <IubendaProvider bannerConfig={iubendaBannerConfig}>
            // Any component at this level will have access to useIubenda()
            {children}
          </IubendaProvider>
        </body>
      </html>
    </>
  );
}
```

Now you can simply wrap any components with the `<ConsentAwareWrapper />` provided by the library or access the `useIubenda()` directly.

## Writing consent-aware components

In some cases you just need to toggle some components based on some requirments. This can be easily achieved using the `<ConsentAwareWrapper />`.

```tsx
// ./app/page.tsx

import { ConsentAwareWrapper } from '@mep-agency/next-iubenda';

// ...
export default function Home() {
  return (
    <main>
      {/* ... */}
      <ConsentAwareWrapper requiredGdprPurposes={['experience', 'marketing']}>
        {/* Anything at this level won't be rendered unless the required permissions have been granted */}
        Both "experience" and "marketing" cookies can be used here!
      </ConsentAwareWrapper>
      {/* // ... */}
    </main>
  );
}
// ...
```

## Advanced features

If you need more flexibility then you can use the `useIubenda()` hook directly:

```tsx
// ./components/ConsentAwareComponent.tsx

'use client';

import { useIubenda } from '@mep-agency/next-iubenda';

const ConsentAwareComponent = () => {
  const {
    userPreferences, // The latest available user preferences data
    showCookiePolicy, // Displays the cookie policy popup
    openPreferences, // Opens the preferences panel
    showTcfVendors, // Opens the TCF vendors panel
    resetCookies, // Resets all cookies managed by Iubenda

    /*
     * The following exposed entries are meant for internal use only and should
     * not be used in your projects.
     */
    dispatchUserPreferences, // Update the user preferences data across the app
    i18nDictionary, // Contains the translations for the built-in components
  } = useIubendaConsent();

  return (
    <div>
      User preferences status:{' '}
      <span style={{ color: userPreferences.hasBeenLoaded ? 'green' : 'red' }}>
        {userPreferences.hasBeenLoaded ? 'LOADED' : 'LOADING...'}
      </span>
    </div>
  );
};

export default ConsentAwareComponent;
```
