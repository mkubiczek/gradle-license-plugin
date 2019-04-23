# Gradle License Plugin

[![License](https://img.shields.io/badge/license-apache%202.0-blue.svg)](http://www.apache.org/licenses/LICENSE-2.0)
[![Build Status](https://travis-ci.org/jaredsburrows/gradle-license-plugin.svg?branch=master)](https://travis-ci.org/jaredsburrows/gradle-license-plugin)
[![Twitter Follow](https://img.shields.io/twitter/follow/jaredsburrows.svg?style=social)](https://twitter.com/jaredsburrows)

This plugin provides a task to generate a HTML license report based on the 
configuration. (eg. `licenseDebugReport` for all debug dependencies in an Android project).

Applying this to an Android or Java project will generate a the license 
file(`open_source_licenses.html`) in the `<project>/build/reports/licenses/`.

Also, for Android projects the license HTML file will be copied to `<project>/src/main/assets/`.

## Download

**Release:**
```groovy
buildscript {
  repositories {
    jcenter()
  }

  dependencies {
    classpath 'com.jaredsburrows:gradle-license-plugin:0.8.5'
  }
}

apply plugin: 'com.android.application' // or 'java-library'
apply plugin: 'com.jaredsburrows.license'
```
Release versions are available in the [JFrog Bintray repository](https://jcenter.bintray.com/).

**Snapshot:**
```groovy
buildscript {
  repositories {
    maven { url 'https://oss.jfrog.org/artifactory/oss-snapshot-local/' }
  }

  dependencies {
    classpath 'com.jaredsburrows:gradle-license-plugin:0.8.6-SNAPSHOT'
  }
}

apply plugin: 'com.android.application' // or 'java-library'
apply plugin: 'com.jaredsburrows.license'
```
Snapshot versions are available in the [JFrog Artifactory repository](https://oss.jfrog.org/artifactory/libs-snapshot/).

## Tasks

- **`license${variant}Report`** for Android
- **`licenseReport`** for Java

Generates a HTML report of the all open source licenses. (eg. `licenseDebugReport` for all debug dependencies in an Android project).

Example `build.gradle`:

```groovy
dependencies {
  compile 'com.android.support:design:26.1.0'
  compile 'pl.droidsonroids.gif:android-gif-drawable:1.2.3'
  compile 'wsdl4j:wsdl4j:1.5.1' // Very old library with no license info available
}
```

**HTML:**
```html
<html>
  <head>
    <style>body { font-family: sans-serif } pre { background-color: #eeeeee; padding: 1em; white-space: pre-wrap; display: inline-block }</style>
    <title>Open source licenses</title>
  </head>
  <body>
    <h3>Notice for packages:</h3>
    <ul>
      <li>
        <a href="#76480">WSDL4J</a>
      </li>
      <pre>No license found</pre>
      <li>
        <a href="#1068328410">Android GIF Drawable Library</a>
      </li>
      <a name="1068328410" />
      <pre>mit.txt here</pre>
      <li>
        <a href="#314129783">Design</a>
      </li>
      <a name="314129783" />
      <pre>apache-2.0.txt here</pre>
    </ul>
  </body>
</html>
```

**JSON:**
```json
[
  {
    "project": "Android GIF Drawable Library",
    "description": "Views and Drawable for displaying animated GIFs for Android",
    "version": "1.2.3",
    "developers": [
      "Karol Wrótniak"
    ],
    "url": "https://github.com/koral--/android-gif-drawable",
    "year": null,
    "licenses": [
      {
        "license": "The MIT License",
        "license_url": "http://opensource.org/licenses/MIT"
      }
    ],
    "dependency": "pl.droidsonroids.gif:android-gif-drawable:1.2.3"
  },
  {
    "project": "Design",
    "description": null,
    "version": "26.1.0",
    "developers": [
        
    ],
    "url": null,
    "year": null,
    "licenses": [
      {
        "license": "The Apache Software License",
        "license_url": "http://www.apache.org/licenses/LICENSE-2.0.txt"
      }
    ],
    "dependency": "com.android.support:design:26.1.0"
  },
  {
    "project": "WSDL4J",
    "description": "Java stub generator for WSDL",
    "version": "1.5.1",
    "developers": [

    ],
    "url": "http://sf.net/projects/wsdl4j",
    "year": null,
    "licenses": [

    ],
    "dependency": "wsdl4j:wsdl4j:1.5.1"
  }
]
```

Note, if no license information is found for a component, the `licenses` element in the JSON output will be an empty array.

## Configuration
The plugin can be configured to generate specific reports and automatically copy the reports to the assets directory (Android projects only). The default behaviours are: 
- Java projects: Generate both the HTML report and the JSON report.
- Android projects: Generate both the HTML report and the JSON report, and copy the HTML report to the assets directory.

To override the defaults, add the `licenseReport` configuration closure to the build script.

```groovy
apply plugin: "com.jaredsburrows.license"

licenseReport {
    generateHtmlReport = false
    generateJsonReport = true
    
    // These options are ignored for Java projects
    copyHtmlReportToAssets = true
    copyJsonReportToAssets = false
}
```

The `copyHtmlReportToAssets` option in the above example would have no effect since the HTML report is disabled.

## Usage

### Create an open source dialog
```java
public static class OpenSourceLicensesDialog extends DialogFragment {

  public OpenSourceLicensesDialog() {
  }

  @Override public Dialog onCreateDialog(@Nullable Bundle savedInstanceState) {
    WebView webView = new WebView(getActivity());
    webView.loadUrl("file:///android_asset/open_source_licenses.html");

    return new AlertDialog.Builder(requireActivity())
      .setTitle("Open Source Licenses")
      .setView(webView)
      .setPositiveButton(R.string.ok, new DialogInterface.OnClickListener() {
        @Override public void onClick(DialogInterface dialog, int which) {
          dialog.dismiss();
        }
      }
    )
    .create();
  }
}
```

### How to use it
```java
public static void showOpenSourceLicenses(Activity activity) {
  FragmentManager fragmentManager = activity.getFragmentManager();
  FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();
  Fragment previousFragment = fragmentManager.findFragmentByTag("dialog_licenses");
  if (previousFragment != null) {
    fragmentTransaction.remove(previousFragment);
  }
  fragmentTransaction.addToBackStack(null);

  new OpenSourceLicensesDialog().show(fragmentTransaction, "dialog_licenses");
}
```

Source: https://github.com/google/iosched/blob/2531cbdbe27e5795eb78bf47d27e8c1be494aad4/android/src/main/java/com/google/samples/apps/iosched/util/AboutUtils.java#L52

<img src="https://www.bignerdranch.com/assets/img/blog/2015/07/screenshot-gmail.png" />

Source: https://www.bignerdranch.com/blog/open-source-licenses-and-android/

## License

    Copyright (C) 2016 Jared Burrows

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.
