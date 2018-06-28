# React Libraries structure
For components publishing and project structures, the library to use will be `lerna`.
~~~
npm in -g lerna
~~~
For initialize lerna configuration, that specifies the package version and which folders to scan for their publication:
~~~
lerna init --independent
~~~
The `--independent` parameter allows to manage different version for each package. It must generate two files: `package.json` and `lerna.json`, and a folder `packages`.

## Lerna configuration
The file `lerna.json` was structured:
```json
{
  "lerna": "2.11.0",
  "packages": [
    "packages/*"
  ],
  "version": "independent"
}
```

The `packages` configuration is a list of folders to scan.
And the `version` configuration specifies how lerna manages the version for internal publish, the other valid values are the global version for all packages.

### Manage dependencies
Lerna allows the dependency manager for the internal packages.

The main structure is:
~~~
lerna add <package-name (internal or external)>[@version] [--scope=<local_package_name>] [--dev]
~~~

If the `scope` parameter is not defined, it install the package globally (all packages).

### Update packages dependencies
Lerna `bootstrap` executes a command that update and install the dependencies in the packages, and updates the dependencies between the packages of the project, and prepares the packages to publish them.

If the `npm install` command (used internally) requires additional arguments, they can be specified in the `lerna.json` file:

```json
{
    ...
    "npmClientArgs": ["--production", "--no-optional"]
}
```

## Package definition
The packages must be defined inside a folder configured in the `lerna.json`. Each package must contain a `package.json` file that identifies the folder as package. Lerna identifies the packages automatically. If there is not that file, you can generate it with `npm init [--scope=<scope_name>]` command.

## Configure React component
If you want to generate a React component, there are additional configurations about `webpack` and `babel`, including internationalization support.

### Configure `babel`
In the package folder add the file `.babelrc`, add the following lines:

```json
{
    "presets": [
        "env"
    ],
    "plugins": [
        "transform-object-rest-spread",
        "transform-react-jsx"
    ]
}
```

### Configure `webpack`
The basic structure for the webpack compilation process, must add the file `webpack.config.js`, and add the next code:

```javascript
var path = require('path');

module.exports = {
    entry: './src/index.js',
    output: {
        path: path.resolve(__dirname, 'build'),
        filename: 'index.js',
        libraryTarget: 'commonjs2'
    },
    module: {
        rules: [
            {
                test: /\.js$/,
                include: path.resolve(__dirname, 'src'),
                exclude: /(node_modules|bower_components|build)/,
                use: {
                    loader: 'babel-loader',
                    options: {
                        presets: ['env']
                    }
                }
            }
        ]
    },
    externals: {
        'react': 'commonjs react'
    }
};
```

### Configure `package.json`
Inside the `package.json` file, you must configure: The main file that will be used when the package is imported, the dependencies, and the script commands to generate file and allows live editing.

1. Add a new item:

```json
{
    ...
    "main": "build/index.js",
    ...
}
```
Note: The `build` folder is created when the script build is called.

2. Add the dependencies

```json
{
    ...
    "peerDependencies": {
        "react": "^16.4.1"
    },
    "dependencies": {
        "babel-polyfill": "6.26.0",
        "lodash": "4.17.10",
        "react": "^16.4.1",
    },
    "devDependencies": {
        "babel-cli": "6.26.0",
        "babel-core": "6.26.3",
        "babel-loader": "7.1.4",
        "babel-plugin-dynamic-import-node": "1.2.0",
        "babel-plugin-react-transform": "3.0.0",
        "babel-plugin-transform-es2015-modules-commonjs": "6.26.2",
        "babel-plugin-transform-react-constant-elements": "6.23.0",
        "babel-plugin-transform-react-inline-elements": "6.22.0",
        "babel-plugin-transform-react-jsx-source": "6.22.0",
        "babel-plugin-transform-react-remove-prop-types": "0.4.13",
        "babel-preset-env": "1.7.0",
        "babel-preset-react": "6.24.1",
        "babel-preset-stage-0": "6.24.1",
        "webpack": "4.12.0",
        "webpack-cli": "3.0.8"
    },
    ...
}
```

If require specific configurations, you must add those lines:

#### Internationalization

```json
{
    ...
    "dependencies": {
        ...
        "intl": "1.2.5",
        "react-intl": "2.4.0",
        ...
    },
    "devDependencies": {
        ...
        "babel-plugin-react-intl": "2.4.0",
        ...
    }
    ...
}
```

#### Styled components

```json
{
    ...
    "dependencies": {
        ...
        "styled-components": "3.3.2",
        ...
    },
    "devDependencies": {
        ...
        "babel-plugin-styled-components": "1.5.1",
        ...
    }
    ...
}
```

Or it can be added using the `npm install` or `lerna add` commands.

3. Add compilation script

The `start` script, run a live changes detector and update the compiled files. It is usefull when you are testing the component in dev mode.

The `build` script, compiles and generate the file in the `build` folders.

```json
{
    ...
    "scripts": {
        ...
        "start": "webpack --watch",
        "build": "webpack"
    }
    ...
}
```

### Adding dependencies and bootstrap the packages
For external dependencies, you would add the packages in the `package.json` dependencies or devDependencies. If the dependencies are internal, is recommended use the `lerna add` commands.

After you finish the configuration, you can use `lerna bootstrap` command. It installs the external dependencies in the `node_modules` folder in each package.

### Running and linking packages
To convert the package as a shared library between our packages and test applications, inside the folder of the package, use the next code:

~~~
npm run build # Generate the main file of the component
~~~

~~~
npm link # Generate the link of the package in npm repo. Use sudo if generate access error
~~~

In the target app, for testing purpouses, we can link the package by the name:

~~~
npm link <package_name> # Obtain the project link registered before
~~~

If you want to develop and check the changes in real time, you can run `npm run start` from package folder, and its build and refresh the main file used by the linked app.

### Publish package
Before realize the publication of a new versions in the npm package manager, you must login:

~~~
npm login
~~~

You already have created a npmjs.com account.

If you want to register a scoped package, you need the domain (if the scope is personal you must use the default domain https://registry.npmjs.org/)

~~~
npm config set scope @<scope>:registry <domain>
~~~

If you want to publish your organization scope in a public context (This is mandatory when you don't have a paid npmjs subscription):

~~~
npm config set access public
~~~

To publish and review the changes of the version, you must user the `lerna publish` command. It guides you through questions and evaluate if the packages needs and upgrade, and simplifies the `npm publish` process.
