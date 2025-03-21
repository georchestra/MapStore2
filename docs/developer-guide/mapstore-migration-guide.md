# Migration Guidelines

## General update checklist

- updating an existing installation
- updating a MapStore project created for a previous version

To update an existing installation you usually have to do nothing except eventually to execute queries on your database to update the changes to the database schema.

In case of a project it becomes a little more complicated, depending on the customization.

This is a list of things to check if you want to update from a previous version valid for every version.

- Take a list to migration notes below for your version
- Take a look to the release notes
- update your `package.json` to latest libs versions
- take a look at your custom files to see if there are some changes (e.g. `localConfig.js`, `proxy.properties`)
- Some changes that may need to be ported could be present also in `pom.xml` files and in `configs` directory.
- check for changes also in `web/src/main/webapp/WEB-INF/web.xml`.
- Optionally check also accessory files like `.eslinrc`, if you want to keep aligned with lint standards.
- Follow the instructions below, in order, from your version to the one you want to update to.


## Migration from 2021.02.00 to 2022.01.00

This release includes several libraries upgrade on the backend side,
in particular the following have been migrated to the latest available versions:

| Library      | Old |New|
| ----------- | ----------- |--|
| Spring| 3.0.5|5.3.9|
| Spring-security| 3.0.5        |5.3.10|
| CXF| 2.3.2        |3.4.4|
| Hibernate|      3.3.2   |5.5.0|
| JPA| 1.0        |2.1|
| hibernate-generic-dao| 0.5.1        |1.3.0-SNAPSHOT|
| h2| 1.3.168        |1.3.175|
| servlet-api| 2.5        |3.1|

This requires also an upgrade of Tomcat to at least version 8.5.x

### Updating projects configuration

Projects need the following to update to this MapStore release:
 
 - update dependencies (in web/pom.xml) copying those in MapStore2/java/web/pom.xml, in particular:

| Dependency      | Version| Notes |
| ----------- | ----------- |---|
| mapstore-services| 1.3-SNAPSHOT| Replaces mapstore-backend|
| geostore-webapp| 1.8-SNAPSHOT| |
| javax.servlet-api| 3.0.1        | Tomcat 8.5 required|


 - update packagingExcludes in web/pom.xml to this list:

```
WEB-INF/lib/commons-codec-1.2.jar,
WEB-INF/lib/commons-io-1.1.jar,
WEB-INF/lib/commons-logging-1.0.4.jar,
WEB-INF/lib/commons-pool-1.3.jar,
WEB-INF/lib/slf4j-api-1.5*.jar,
WEB-INF/lib/slf4j-log4j12-1.5*.jar,
WEB-INF/lib/spring-tx-5.2.15*.jar
```

 - upgrade Tomcat to 8.5 or greater
 - update your geostore-spring-security.xml file to add the following setting, needed to disable CSRF validation, that MapStore services do not implement yet:

```xml
<security:http ... >
    ...
    <security:csrf disabled="true"/>
    ...
</security:http>
```

 - remove the spring log4j listener from web.xml

```xml
 <!-- spring context loader
    <listener>
		<listener-class>org.springframework.web.util.Log4jConfigListener</listener-class>
    </listener>-->
```

### Upgrading CesiumJS
CesiumJS has been upgraded to version 1.42 (from 1.17), with no breaking changes, so you should replace all the instances of  `https://cesium.com/downloads/cesiumjs/releases/1.17` in your projects HTML files with `https://cesium.com/downloads/cesiumjs/releases/1.42`. The new release is needed to implement Bearer based authentication to WMS services.

## Migration from 2021.01.04 to 2021.02.00

### Theme updates and CSS variables

The theme of MapStore has been updated to support CSS variables for some aspects of the style, in particular colors and font families.
The `web/client/themes/default/variables.less` file contains all the available variables described under the `@ms-theme-vars` ruleset.
It is suggested to :

- update the lesscss variables in the projects because the variables starting with `@ms2-` will be deprecated soon:

`@ms2-color-text` -> `@ms-main-color`
`@ms2-color-background` -> `@ms-main-bg`
`@ms2-color-shade-lighter` -> `@ms-main-border-color`

`@ms2-color-code` -> `@ms-code-color`

`@ms2-color-text-placeholder` -> `@ms-placeholder-color`

`@ms2-color-disabled` -> `@ms-disabled-bg`
`@ms2-color-text-disabled` -> `@ms-disabled-color`

`@ms2-color-text-primary` -> `@ms-primary-contrast`

`@ms2-color-primary` -> `@ms-primary`
`@ms2-color-info` -> `@ms-info`
`@ms2-color-success` -> `@ms-success`
`@ms2-color-warning` -> `@ms-warning`
`@ms2-color-danger` -> `@ms-danger`

- The font family has been update to `Noto Sans` so all the html need to be updated removing the previous font link with:

```html
<link rel="preconnect" href="https://fonts.gstatic.com">
<link href="https://fonts.googleapis.com/css2?family=Noto+Sans&display=swap" rel="stylesheet">
```

- if you are importing `react-select` or `react-widgets` inline css/less in your own project, you have to remove the import. Now the stile of these libraries is managed at project level

### Project system

During this release MapStore we started an rewrite of the [project system](https://github.com/geosolutions-it/MapStore2/issues/6314), organized in different phases.

The first phase of this migration has been identified by [this](https://github.com/geosolutions-it/MapStore2/pull/6738) pull request. In this phase we are supporting the backward compatibility as much as possible, introducing the new project system in parallel with the new one (experimental).
In the future the current script will be deprecated in favor of the new one.

Here below the breaking changes introduced in this release to support this new system:

#### The following key files have been moved to the new `configs` folder

We suggest you to move them as well from root to configs folder

- `localConfig.json`
- `new.json`
- `pluginsConfig.json`
- `config.json`
- `simple.json`

#### Back-end has been reorganized

In particular:

- all the java code has been moved from `web/src/` to the `java/` and `product/` directories (and `release`, already existing).
- `mapstore-backend` has been renamed into `mapstore-services`.
- `geostore` version used is `1.7.0` on release `2021.02.00`

Check the changes in `pom.xml` to update. (future evolution of the project will avoid you to keep your own copies of the pom files as much as possible, for this reasons these migration guidelines will change soon.)

- `pom.xml`
- `java/web/pom.xml`


#### Data directory has been reorganized and is now available also for product

The new organization of the data directory is:

- `configs` will contain all json files (`localConfig.json`, `new.json`, `pluginsConfig.json`, ...) and all the `.patch` files applied to them.
- `extensions` folder contains all the data for the extensions, including `extensions.json`
- The root contains the properties files to configure database, proxy and other configs

To organize your old data directory accordingly to the new specification.

- Move all `.json` and `.json.patch` files in `configs` folder (except `extensions.json`)
- Move the directory `dist/extensions` to simply `extensions`.
- The file `extensions.json` have to be moved in `extensions/extensions.json`.
- Edit the file `extensions/extensions.json` changing the paths from `dist/extensions/<Plugin-Name>/...` to `<Plugin-Name>/...`

You can set it up by configuring `datadir.location` java system property. Changes to paths or configuration files are not required anymore.

### Configurations

- Embedded now uses popup as default. Align localConfig.json `plugins --> embedded --> Identify` with the latest one:

```json
{
    "name": "Identify",
    "cfg": {
        "showInMapPopup":true,
        "viewerOptions": {
            "container": "{context.ReactSwipe}"
        }
    }
}
```

## Migration from 2021.01.01 to 2021.01.03

Generally speaking this is not properly a breaking change, but more a fix to apply to your installations. Certificate for 'cesiumjs.org' has expired at 2021.05.05, so to continue using CesiumJS with MapStore
you will have to replace all the URLs like `https://cesiumjs.org/releases/1.17` in `https://cesium.com/downloads/cesiumjs/releases/1.17`. This is the main fix of this minor release.
See [this pull request on GitHub](https://github.com/geosolutions-it/MapStore2/pull/6856) as a sample to apply these changes to your project.

## Migration from 2021.01.00 to 2021.01.01

### Update embedded entry to load the correct configuration

Existing MapStore project could have an issue with the loading of map embedded page due to the impossibility to change some configuration such as localConfig.json or translations path in the javascript entry.
This issue can be solved following these steps:
1 - add a custom entry named `embedded.jsx` in the `js/` directory of the project with the content:
```js
import {
    setConfigProp,
    setLocalConfigurationFile
} from '@mapstore/utils/ConfigUtils';

// Add custom (overriding) translations
// example for additional translations in the project folder
// setConfigProp('translationsPath', ['./MapStore2/web/client/translations', './translations']);
setConfigProp('translationsPath', './MapStore2/web/client/translations');
// __PROJECTNAME__ is the name of the project used in the creation process
setConfigProp('themePrefix', '__PROJECTNAME__');

// Use a custom plugins configuration file
// example if localConfig.json is located in the root of the project
// setLocalConfigurationFile('localConfig.json');
setLocalConfigurationFile('MapStore2/web/client/localConfig.json');

// async load of the standard embedded bundle
import('@mapstore/product/embedded');
```
2 - update the path of the embedded entry inside the `webpack.config.js` and `prod-webpack.config.js` files with:
```js
// __PROJECTNAME__ is the name of the project used in the creation process
'__PROJECTNAME__-embedded': path.join(__dirname, "js", "embedded"),
```
### Locate plugin configuration

Configuration for Locate plugin has changed and it is not needed anymore inside the Map plugin

- old localConfig.json configuration needed 'locate' listed as tool inside the Map plugin and as a separated Locate plugin
```js
// ...
{
    "name": "Map",
    "cfg": {
        "tools": ["locate"],
        // ...
    }
},
{
    "name": "Locate",
    // ...
}
// ...
```

- new localConfig.json configuration removes 'locate' from tools array and it keeps only the plugin configuration
```js
// ...
{
    "name": "Map",
    "cfg": {
        // ...
    }
},
{
    "name": "Locate",
    // ...
}
// ...
```

### Update an existing project to include embedded Dashboards and GeoStories

Embedded Dashboards and GeoStories need a new set of javascript entries, html templates and configuration files to make them completely available in an existing project.

The steps described above assume this structure of the MapStore2 project for the files that need update:

```
MapStore2Project/
|-- ...
|-- js/
|    |-- ...
|    |-- dashboardEmbedded.jsx (new)
|    |-- geostoryEmbedded.jsx (new)
|-- MapStore2/
|-- web/
|    |-- ...
|    |-- pom.xml
|-- ...
|-- dashboard-embedded-template.html (new)
|-- dashboard-embedded.html (new)
|-- ...
|-- geostory-embedded-template.html (new)
|-- geostory-embedded.html (new)
|-- ...
|-- prod-webpack.config.js
|-- ...
|-- webpack.config.js
```

1) create the entries files for the embedded application named `dashboardEmbedded.jsx` and `geostoryEmbedded.jsx` in the js/ folder with the following content (see links):
    - [dashboardEmbedded.jsx](https://github.com/geosolutions-it/MapStore2/blob/2021.01.xx/project/standard/templates/js/dashboardEmbedded.jsx)
    - [geostoryEmbedded.jsx](https://github.com/geosolutions-it/MapStore2/blob/2021.01.xx/project/standard/templates/js/geostoryEmbedded.jsx)

2) add the html files and templates in the root directory of the project with these names and content (see links):
    - [dashboard-embedded-template.html](https://github.com/geosolutions-it/MapStore2/blob/2021.01.xx/project/standard/templates/dashboard-embedded-template.html)
    - [dashboard-embedded.html](https://github.com/geosolutions-it/MapStore2/blob/2021.01.xx/project/standard/templates/dashboard-embedded.html)
    - [geostory-embedded-template.html](https://github.com/geosolutions-it/MapStore2/blob/2021.01.xx/project/standard/templates/geostory-embedded-template.html)
    - [geostory-embedded.html](https://github.com/geosolutions-it/MapStore2/blob/2021.01.xx/project/standard/templates/geostory-embedded.html)

3) update webpack configuration for development and production with the new entries and the related configuration:

    - webpack.config.js
    ```js
    module.exports = require('./MapStore2/build/buildConfig')(
        {
            // other entries...,
            // add new embedded entries to entry object
            "geostory-embedded": path.join(__dirname, "js", "geostoryEmbedded"),
            "dashboard-embedded": path.join(__dirname, "js", "dashboardEmbedded")
        },
        // ...
    );
    ```
    - prod-webpack.config.js

    ```js

    module.exports = require('./MapStore2/build/buildConfig')(
        {
            // other entries...,
            // add new embedded entries to entry object
            "geostory-embedded": path.join(__dirname, "js", "geostoryEmbedded"),
            "dashboard-embedded": path.join(__dirname, "js", "dashboardEmbedded")
        },
        // ...
        [
            // new HtmlWebpackPlugin({ ... }),
            // add plugin to copy all the embedded html and inject the correct bundle
            new HtmlWebpackPlugin({
                template: path.join(__dirname, 'geostory-embedded-template.html'),
                chunks: ['geostory-embedded'],
                inject: "body",
                hash: true,
                filename: 'geostory-embedded.html'
            }),
            new HtmlWebpackPlugin({
                template: path.join(__dirname, 'dashboard-embedded-template.html'),
                chunks: ['dashboard-embedded'],
                inject: 'body',
                hash: true,
                filename: 'dashboard-embedded.html'
            })
        ],
        // ...
    );
    ```

4) Add configuration to localConfig.json in the plugins section related to Share functionalities (Only with custom localConfig.json in the project):
    - Dashboard share configuration
    ```js
    "dashboard": [
        // ...
        {
            "name": "Share",
            "cfg": {
                "showAPI": false,
                "advancedSettings": false,
                "shareUrlRegex": "(h[^#]*)#\\/dashboard\\/([A-Za-z0-9]*)",
                "shareUrlReplaceString": "$1dashboard-embedded.html#/$2",
                "embedOptions": {
                    "showTOCToggle": false,
                    "showConnectionsParamToggle": true
                }
            }
        },
        // ...
    ]
    ```

    - Dashboard share configuration
    ```js
    "geostory": [
        // ...
        {
            "name": "Share",
            "cfg": {
                "embedPanel": true,
                "showAPI": false,
                "advancedSettings": {
                    "hideInTab": "embed",
                    "homeButton": true,
                    "sectionId": true
                },
                "shareUrlRegex": "(h[^#]*)#\\/geostory\\/([^\\/]*)\\/([A-Za-z0-9]*)",
                "shareUrlReplaceString": "$1geostory-embedded.html#/$3",
                "embedOptions": {
                    "showTOCToggle": false
                }
            }
        },
        // ...
    ]
    ```

5) update the web/pom.xml to copy all the related resources in the final *.war file with these new executions
```xml
<!-- __PROJECTNAME__ should be equal to the one in use in the project, see other executions how they define the outputDirectory path  -->
<execution>
    <id>only dashboard-embedded.html</id>
    <phase>process-classes</phase>
    <goals>
        <goal>copy-resources</goal>
    </goals>
    <configuration>
        <outputDirectory>${basedir}/target/__PROJECTNAME__</outputDirectory>
        <encoding>UTF-8</encoding>
        <resources>
            <resource>
                <directory>${basedir}/../dist</directory>
                <includes>
                    <include>dashboard-embedded.html</include>
                </includes>
                <excludes>
                    <exclude>MapStore2/*</exclude>
                    <exclude>MapStore2/**/*</exclude>
                </excludes>
            </resource>
        </resources>
    </configuration>
</execution>
<execution>
    <id>only geostory-embedded.html</id>
    <phase>process-classes</phase>
    <goals>
        <goal>copy-resources</goal>
    </goals>
    <configuration>
        <outputDirectory>${basedir}/target/__PROJECTNAME__</outputDirectory>
        <encoding>UTF-8</encoding>
        <resources>
            <resource>
                <directory>${basedir}/../dist</directory>
                <includes>
                    <include>geostory-embedded.html</include>
                </includes>
                <excludes>
                    <exclude>MapStore2/*</exclude>
                    <exclude>MapStore2/**/*</exclude>
                </excludes>
            </resource>
        </resources>
    </configuration>
</execution>
```

## Migration from 2020.02.00 to 2021.01.00

### Update to webpack 5 - Module federation

MapStore migrated to webpack 5 and provided the extension system using "Webpack Module Federation". Here the steps to update the existing files in your project.

**package.json**:

- dev server scripts changed syntax. now you need to use `webpack serve` instead of `webpack-dev-server`. Replace also all `--colors` with `--color` in your scripts that use webpack / webpack-dev-server.
- Align `dependencies` and `devDependencies` with MapStore's one, reading the `package.json`, as usual.
- To support extensions in your project, you need to add `ModuleFederationPlugin` to your `prod-webpack.config.js` and `webpack.config.js`

```javascript
const ModuleFederationPlugin = require('./MapStore/build/moduleFederation').plugin; // <-- new line
module.exports = require('./buildConfig')(
    assign({
        "mapstore2": path.join(paths.code, "product", "app"),
        "embedded": path.join(paths.code, "product", "embedded"),
        "ms2-api": path.join(paths.code, "product", "api")
    },
    require('./examples')
    ),
    themeEntries,
    paths,
    extractThemesPlugin,
    [extractThemesPlugin, ModuleFederationPlugin], // <-- this parameter has been changed, now it accepts also array of the plugins you want to add bot in prod and dev
```

Other the other changes required are applied automatically in `buildConfig.js`.

### Eslint config

Now eslint configuration is shared in a separate npm module. To update your custom project you have to remove the following files:

- `.eslintignore`
- `.eslintconfig`

And add to `package.json` the following entry, in the root:

```json
        "eslintConfig": {
            "extends": [
                "@mapstore/eslint-config-mapstore"
            ],
            "parserOptions": {
                "babelOptions": {
                    "configFile": "./MapStore2/build/babel.config.js"
                }
            }
        },
```

If you have aproject that includes MapStore as a dependency, you can run `npm run updateDevDeps` to finalize the update. Otherwise make you sure to include:

- devDependencies:
  - add `"@mapstore/eslint-config-mapstore": "1.0.1",`
  - delete `babel-eslint`
- dependencies:
  - update `"eslint": "7.8.1"


### App structure review

From this version some base components of MapStore App (`StandardApp`, `StandardStore`...) has been restructured and better organized. Here a list of the breaking change you can find in a depending project

- `web/client/product/main.jsx` has been updated to new `import` and `export` syntax (removed `require` and `exports.module`). So if you are importing it (usually in your `app.jsx`) you have to use the `import` syntax or use `require(...).default` in your project. The same for the other files.
- New structure of arguments in web/client/stores/StandardStore.js

```js
const appStore = (
    {
        initialState = {
            defaultState: {},
            mobile: {}
        },
        appReducers = {},
        appEpics = {},
        rootReducerFunc = ({ state, action, allReducers }) => allReducers(state, action)
    },
    plugins = {},
    storeOpts = {}
) {
  ...
```

- Moved standard epics, standard reducers and standard rootReducer function from web/client/stores/StandardStore.js to a separated file web/client/stores/defaultOptions.js


- loading extensions functionalities inside StandardApp has been moved to an specific withExtensions HOC, so if you are not using `main.js` but directly `StandardApp` and you need extensions you need to add this HOC to your StandardApp

## Migration from 2020.01.00 to 2020.02.00

### New authentication rule for internal services

With this new version the support for uploading extensions has been introduced. A new entry point needs administration authorization to allow the upload of new plugins by the administrator. So:

- In `localConfig.json` add the following entry in the `authenticationRules` array:
```json
{
    "urlPattern": ".*rest/config.*",
    "method": "bearer"
  }

```
the final entry should look like this 

```json
 "authenticationRules": [{
        "urlPattern": ".*geostore.*",
        "method": "bearer"
      }, {
        "urlPattern": ".*rest/config.*",
        "method": "bearer"
      }, ...],
```

### Translation files

- The translations file extension has been changed into JSON. Now translation files has been renamed from `data.<locale>` to `data.<locale>.json`. This change makes the `.json` extension mandatory for all translation files. This means that depending projects with custom translation files should be renabled in the same name. E.g. `data.it-IT` have to be renamed as `data.it-IT.json`

### Database Update

Database schema has changed. To update your database you need to apply this SQL scripts to your database

- Update the user schema
run the script available [here](https://github.com/geosolutions-it/geostore/tree/master/doc/sql/migration/postgresql):
```sql

-- Update the geostore database from 1.4.2 model to 1.5.0
-- It adds fields to gs_security for external authorization

-- The script assumes that the tables are located into the schema called "geostore"
--      if you put geostore in a different schema, please edit the following search_path.
SET search_path TO geostore, public;

-- Tested only with postgres9.1

-- Run the script with an unprivileged application user allowed to work on schema geostore

alter table gs_security add column username varchar(255);
alter table gs_security add column groupname varchar(255);

create index idx_security_username on gs_security (username);

create index idx_security_groupname on gs_security (groupname);

```


- Add new categories

```sql
-- New CONTEXT category
INSERT into geostore.gs_category (id ,name) values ( nextval('geostore.hibernate_sequence'),  'CONTEXT') ON CONFLICT DO NOTHING;
-- New GEOSTORY category (introduced in 2020.01.00)
INSERT into geostore.gs_category (id ,name) values (nextval('geostore.hibernate_sequence'),  'GEOSTORY') ON CONFLICT DO NOTHING;
-- New TEMPLATE category
INSERT into geostore.gs_category (id ,name) values ( nextval('geostore.hibernate_sequence'),  'TEMPLATE') ON CONFLICT DO NOTHING;
-- New USERSESSION category
INSERT into geostore.gs_category (id ,name) values ( nextval('geostore.hibernate_sequence'),  'USERSESSION') ON CONFLICT DO NOTHING;

```

### Backend update
For more details see [this](https://github.com/geosolutions-it/MapStore2/commit/4aa7b917abcb09571af5b9999a38e96f52eac4f3#diff-ac81cff563b78256ef26eca8a5103392592c7138987392a6fb3d79167d11bdcfR66) commit

new files have been added:

-  `web/src/main/webapp/WEB-INF/dispatcher-servlet.xml`
-  `web/src/main/resources/mapstore.properties`

some files has been changed:

- `web/src/main/webapp/WEB-INF/web.xml`
- `pom.xml`
- `web/pom.xml`


## Migration from 2019.02.01 to 2020.01.00


With MapStore **2020.01.00** some dependencies that were previously hosted on github, have now been published on the npm registry, and package.json has been updated accordingly.
[Here](https://github.com/geosolutions-it/MapStore2/pull/4598) is the PR that documents how to update local package.json and local webpack if not using the mapstore buildConfig/testConfig common files.

After updating package.json run **npm install**
Now you should be able to run locally with **npm start**

For more info see the related [issue](https://github.com/geosolutions-it/MapStore2/issues/4569)

Moreover a new category has been added for future features, called GEOSTORY.

It is not necessary for this release, but, to follow the update sequence, you can add it by executing the following line.

```sql
INSERT into geostore.gs_category (id ,name) values (nextval('geostore.hibernate_sequence'),  'GEOSTORY') ON CONFLICT DO NOTHING;
```

## Migration from 2019.01.00 to 2019.01.01

MapStore **2019.01.01** changes the location of some of the build and test configuration files.
This also affects projects using MapStore build files, sp if you update MapStore subproject to the **2019.01.01** version you also have to update some of the project configuration files. In particular:

- **webpack.config.js** and **prod-webpack.config.js**:
  - update path to themes.js from ./MapStore2/themes.js to ./MapStore2/build/themes.js
  - update path to buildConfig from ./MapStore2/buildConfig to ./MapStore2/build/buildConfig
- **karma.conf.continuous-test.js** and **karma.config.single-run.js**: update path to testConfig from ./MapStore2/testConfig to ./MapStore2/build/testConfig

## Migration from 2017.05.00 to 2018.01.00

MapStore **2018.01.00** introduced theme and js and css versioning.
This allows to auto-invalidates cache files for each version of your software.
For custom projects you could choose to ignore this changes by setting version: "no-version" in your app.jsx `StandardRouter` selector:

```javascript
//...
const routerSelector = createSelector(state => state.locale, (locale) => ({
    locale: locale || {},
    version: "no-version",
    themeCfg: {
        theme: "mythheme"
    },
    pages
}));
const StandardRouter = connect(routerSelector)(require('../MapStore2/web/client/components/app/StandardRouter'));
//...

```

### Support js/theme versioning in your project

Take a look to [this pull request](https://github.com/geosolutions-it/MapStore2/pull/2538/files) as reference.
Basically versioning is implemented in 2 different ways for css and js files :

- Add at build time the js files inclusion to the files, with proper hashes.
- Load theme css files appending to the URL the ?{version} where version is the current mapstore2 version
The different kind of loading for css files is needed to continue supporting the theme switching capabilities.
For the future we would like to unify these 2 systems. See [this issue](https://github.com/geosolutions-it/MapStore2/issues/2554).

You have to:

- Add the version file to the root (`version.txt`).
- Create a template for each html file you have. These files will replace the html files when you build the final  war file. These files are like the original ones but without the `[bundle].js` file inclusion and without theme css.
- Add `HtmlWebpackPlugin` for production only, one for each js file. This plugin will add to the template file the script inclusion ([example](https://github.com/geosolutions-it/MapStore2/pull/2538/files#diff-9d452e0b96db9be8d604c4c9dde575b4)).
  - if you have to include the script in the head (e.g. `api.html` has some script that need the js to be loaded before executing the inline scripts), use the option `inject: 'head'`
- change each entry point (`app.jsx`, `api.jsx`, `embedded.jsx`, `yourcoustomentrypoint.jsx`) this way ([example](https://github.com/geosolutions-it/MapStore2/pull/2538/files#diff-3bea50c2662e64129704ae694b587042)):
  - `version` reducer in `StandardRouter`
  - `loadVersion` action in `initialActions`
  - `version` and `loadAfterTheme` selectors to `StandardRouter` state.

```javascript
// Example
const {versionSelector} = require('../MapStore2/web/client/selectors/version');
const {loadVersion} = require('../MapStore2/web/client/actions/version');
const version = require('../MapStore2/web/client/actions/version');
//...
StandardRouter = connect ( state => ({
    locale: state.locale || {},
        pages,
        version : versionSelector(state),
        loadAfterTheme: loadAfterThemeSelector(state)
    }))(require('../MapStore2/web/client/components/app/StandardRouter'))
const appStore = require('../MapStore2/web/client/stores/StandardStore').bind(null, initialState, {
    // ...
    version: version
});
// ...
const appConfig = {
    // ...
    initialActions: [loadVersion]
}
```

- Add to your `pom.xml` some execution steps to replace html files with the ones generated in 'dist' directory. ([example](https://github.com/geosolutions-it/MapStore2/pull/2538/files#diff-eef89535a29b4a95a42d9de83cb53681)). And copy `version.txt`
- Override the version file in your build process (e.g. you can use the commit hash)


## Migration from 2017.05.00 to 2017.03.00 and previews

In **2017.03.00** the `createProject.js` script created only a custom project. From version 2017.04.00 we changed the script to generate 2 kind of projects:

- **custom**: the previous version
- **standard**: mapstore standard

Standard project wants to help to generate a project that is basically the MapStore product, where you can add your own plugins and customize your theme (before this you had to create a project similar to MapStore on your own)
Depending on our usage of custom project, this may introduce some breaking changes.
If you previously included some file from `product` folder, now `app.jsx` has been changed to call `main.jsx`. Please take a look on how the main product uses this to migrate your changes inside your custom project.

## Migration from 2017.01.00 to 2017.02.00

The version 2017.02.00 has many improvements and changes:

- introduced `redux-observable`
- updated `webpack` to version 2
- updated `react-intl` to version 2.x
- updated `react` to [version 15.4.2] (https://facebook.github.io/react/blog/2016/04/07/react-v15.html)
- updated `react-bootstrap` to version 0.30.7

We suggest you to:

- align your package.json with the latest version of 2017.02.00.
- update your webpack files (see below).
- update your tests to react 15 version. [see upgrade guide](https://facebook.github.io/react/blog/2016/04/07/react-v15.html#upgrade-guide)
- Update your `react-bootstrap` custom components with the new one (see below).

### Side Effect Management - Introduced redux-observable

To manage complex asynchronous operations the thunk middleware is not enough.
When we started with MapStore there was no alternative to thunk. Now we have some options. After a spike (results available [here](https://github.com/geosolutions-it/MapStore2/issues/1420)) we chose to use redux-observable.
For the future, we strongly recommend to use this library to perform asynchronous tasks.

Introducing this library will allow to :

- remove business logic from the components event handlers
- now all new  `actionCreators` should return pure actions. All async stuff will be deferred to the epics.
- avoid bouncing between components and state to trigger side effect
- speed up development with `rxjs` functionalities
- Existing thunk integration will be maintained since all the thunks will be replaced.

If you are using the Plugin system and the StandardStore, you may have only to include the missing new dependencies in your package.json (redux-observable and an updated version of redux).

Check the current package.json to get he most recent versions. For testing we included also redux-mockup-store as a dependency, but you are free to test your epics as you want.

For more complex integrations check [this](https://github.com/geosolutions-it/MapStore2/pull/1471) pull request to see how to integrate redux-observable or follow the guide on the [redux-observable site](https://redux-observable.js.org/).

### Webpack update to version 2

We updated webpack (old one is deprecated), check [this pull request](https://github.com/geosolutions-it/MapStore2/pull/1491) to find out how to update your webpack files.
here a list of what we had to update:

- module.loaders are now module.rules
- update your package.json with latest versions of webpack, webpack plugins and karma libs and integrations (Take a look to the changes on package.json in the pull request if you want a detailed list of what to update in this case).
- change your test proxy configuration with the new one.

More details on the [webpack site](https://webpack.js.org/migrate/).

### react-intl update to  2.x

See [this pull request](https://github.com/geosolutions-it/MapStore2/pull/1495/files) for the details. You should only have to update your package.json

### react update to 15.4.2

Check this pull request to see how to:

- update your package.json
- update your tests

### React Bootstrap update

The version we are using is not documented anymore, and not too much compatible with react 15 (too many warnings). So this update can not be postponed anymore.
The bigger change in this case is that the Input component do not exists anymore. You will have to replace all your Input with the proper components, and update the `package.json`. See [this pull request](https://github.com/geosolutions-it/MapStore2/pull/1511) for details.
