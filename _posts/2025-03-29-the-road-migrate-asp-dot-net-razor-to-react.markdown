---
layout: post
title: "The road migrate ASP.Net Razor to React"
tags:
 -
---

environment: 
```*
macOS
dotnet sdk 8
node 16
```


First step is to create a react project under origin Asp.Net project's root path, like `/ClientApp` to store and isolate Front-end and Back-end codebase clearly.

Creating project can simply execute `npx create-react-app ClientApp`. Then doing some special configuration to enable both can communicate:

Add `PUBLIC_URL` to React index.html.
```html
<meta
      name="description"
      content="Web site created using create-react-app"
/>
<base href="%PUBLIC_URL%/" />
<link rel="apple-touch-icon" href="%PUBLIC_URL%/logo192.png" />
```

Add `"ASPNETCORE_HOSTINGSTARTUPASSEMBLIES": "Microsoft.AspNetCore.SpaProxy"` to ASP.Net `launchSetting.json`.

Create a .env file in ClientApp and add BE port number and HTTPS if necessary(will mention later to configurate)

Then the key part is to create a simple proxy service between React and ASP.Net, we can rely on `http-proxy-middleware` to achieve this.

To doing so, we need to create below file, which will create a proxy middleware along with Front-end startup and keep listening backend API traffics.
The context list item here must match BE Controller names.

```js
const { createProxyMiddleware } = require('http-proxy-middleware');
const { env } = require('process');

const target = env.ASPNETCORE_HTTPS_PORT ? `https://localhost:${env.ASPNETCORE_HTTPS_PORT}` :
  env.ASPNETCORE_URLS ? env.ASPNETCORE_URLS.split(';')[0] : 'http://localhost:14731';

const context = [
  "/NextTrainSchedule",
  "/RampBooking",
  "/GeneralData",
];

const onError = (err, req, resp, target) => {
    console.error(`${err.message}`);
}

module.exports = function (app) {
  const appProxy = createProxyMiddleware(context, {
    proxyTimeout: 10000,
    target: target,
    // Handle errors to prevent the proxy middleware from crashing when
    // the ASP NET Core webserver is unavailable
    onError: onError,
    secure: false,
    // Uncomment this line to add support for proxying websockets
    //ws: true, 
    headers: {
      Connection: 'Keep-Alive'
    }
  });

  app.use(appProxy);
};

```
Then we can creata a simple react component to test if it's work. In below component, I simply call a API to retrieve some BE data and display, the API path can be relative path.

```jsx
import React, { Component } from 'react';

export class FetchData extends Component {
  static displayName = FetchData.name;

  constructor(props) {
    super(props);
    this.state = { forecasts: [], loading: true };
  }

  componentDidMount() {
    this.populateWeatherData();
  }

  static renderForecastsTable(forecasts) {
    return (
      <div>
        {forecasts.map(forecast => <p>{forecast}</p>)}
      </div>
    );
  }

  render() {
    let contents = this.state.loading
      ? <p><em>Loading...</em></p>
      : FetchData.renderForecastsTable(this.state.forecasts);

    return (
      <div>
        <h1 id="tabelLabel" >Weather forecast</h1>
        <p>This component demonstrates fetching data from the server.</p>
        {contents}
      </div>
    );
  }

  async populateWeatherData() {
    const response = await fetch('NextTrainSchedule/GetDestinationCodeList?originStationCode=ADM&funcId=20');
    const data = await response.json();
    this.setState({ forecasts: data, loading: false });
  }
}
```

Then, let's take some challenges to enable HTTPS, since we previously are based on HTTP communication that is not really safe.

There are two parts need to configure,

First we need to create a js script to set up HTTPS for the application using the ASP.NET Core HTTPS certificate, which just leverage dotnet cli's tool to export local https certificate and public key.

```js
const fs = require('fs');
const spawn = require('child_process').spawn;
const path = require('path');

const baseFolder =
  process.env.APPDATA !== undefined && process.env.APPDATA !== ''
    ? `${process.env.APPDATA}/ASP.NET/https`
    : `${process.env.HOME}/.aspnet/https`;

const certificateArg = process.argv.map(arg => arg.match(/--name=(?<value>.+)/i)).filter(Boolean)[0];
const certificateName = certificateArg ? certificateArg.groups.value : process.env.npm_package_name;

if (!certificateName) {
  console.error('Invalid certificate name. Run this script in the context of an npm/yarn script or pass --name=<<app>> explicitly.')
  process.exit(-1);
}

const certFilePath = path.join(baseFolder, `${certificateName}.pem`);
const keyFilePath = path.join(baseFolder, `${certificateName}.key`);

if (!fs.existsSync(certFilePath) || !fs.existsSync(keyFilePath)) {
  spawn('dotnet', [
    'dev-certs',
    'https',
    '--export-path',
    certFilePath,
    '--format',
    'Pem',
    '--no-password',
  ], { stdio: 'inherit', })
  .on('exit', (code) => process.exit(code));
}

```

Then we need the 2nd js script to read and use generated cert and key and store in FE's envrionment file to configure HTTPS using the ASP.NET Core
development certificate in the webpack development proxy.

```js
const fs = require('fs');
const path = require('path');

const baseFolder =
  process.env.APPDATA !== undefined && process.env.APPDATA !== ''
    ? `${process.env.APPDATA}/ASP.NET/https`
    : `${process.env.HOME}/.aspnet/https`;

const certificateArg = process.argv.map(arg => arg.match(/--name=(?<value>.+)/i)).filter(Boolean)[0];
const certificateName = certificateArg ? certificateArg.groups.value : process.env.npm_package_name;

if (!certificateName) {
  console.error('Invalid certificate name. Run this script in the context of an npm/yarn script or pass --name=<<app>> explicitly.')
  process.exit(-1);
}

const certFilePath = path.join(baseFolder, `${certificateName}.pem`);
const keyFilePath = path.join(baseFolder, `${certificateName}.key`);

if (!fs.existsSync('.env.development.local')) {
  fs.writeFileSync(
    '.env.development.local',
`SSL_CRT_FILE=${certFilePath}
SSL_KEY_FILE=${keyFilePath}`
  );
} else {
  let lines = fs.readFileSync('.env.development.local')
    .toString()
    .split('\n');

  let hasCert, hasCertKey = false;
  for (const line of lines) {
    if (/SSL_CRT_FILE=.*/i.test(line)) {
      hasCert = true;
    }
    if (/SSL_KEY_FILE=.*/i.test(line)) {
      hasCertKey = true;
    }
  }
  if (!hasCert) {
    fs.appendFileSync(
      '.env.development.local',
      `\nSSL_CRT_FILE=${certFilePath}`
    );
  }
  if (!hasCertKey) {
    fs.appendFileSync(
      '.env.development.local',
      `\nSSL_KEY_FILE=${keyFilePath}`
    );
  }
}
```

After that, we are done, just launch project with `dotnet run watch` and `npm run start` on seperate terminals to see if they work. Cheers!