# 🎉🎉🎉 Angular Monaco Editor Loader 🎉🎉🎉

[![Greenkeeper badge](https://badges.greenkeeper.io/leolorenzoluis/xyz.MonacoEditorLoader.svg)](https://greenkeeper.io/)
[![npm](https://img.shields.io/badge/awesome-∞-brightgreen.svg)](https://www.npmjs.com/package/@abc.xyz/angular-monaco-editor-loader)
[![Build Status](https://travis-ci.org/leolorenzoluis/xyz.MonacoEditorLoader.svg?branch=master)](https://travis-ci.org/leolorenzoluis/xyz.MonacoEditorLoader)

Easy to install with the following command:
 
```
npm i @abc.xyz/angular-monaco-editor-loader
```


An easy to use Monaco Editor Loader Service for Angular! Just add `*loadMonacoEditor` in your HTML Element, and you don't have to worry about timing issues!

If you are looking for Angular < 4 please see the following branch https://github.com/leolorenzoluis/xyz.MonacoEditorLoader/tree/angular-4

```
<div *loadMonacoEditor id="container"></div> 

```

With custom monaco-editor path
```
<div *loadMonacoEditor="'libs/monaco-editor/vs'" id="container"></div> 

```

# Prerequisites

- Make sure that you are serving `Monaco Editor` in `/assets/monaco-editor/vs`
- If you are using straight `app.component` then **DO NOT USE the directive**. Instead use the following code in `app.component.ts`
```
  constructor(private monaco: MonacoEditorLoaderService) {

  }

  this.monaco.isMonacoLoaded.subscribe(value => {
      if (value) {
        // Initialize monaco...
      }
    })
```
- If you are creating a component on top of monaco, then just use the directive `*loadMonacoEditor` inside your component's HTML
## Using webpack
- If you are using `Webpack` do the following:
```
plugins: [
     new CopyWebpackPlugin([
         {
             from: 'node_modules/monaco-editor/min/vs',
             to: './src/assets/monaco',
         }
     ]),
 ],
 ```
 ## Using Angular CLI
 - Modify `.angular-cli.json` to the following:
 ```
 "assets": [
        {
          "glob": "**/*",
          "input": "../node_modules/monaco-editor/min/",
          "output": "./assets/monaco-editor/"
        },
        {
          "glob": "**/*",
          "input": "../node_modules/monaco-editor/min-maps/",
          "output": "./assets/min-maps/"
        },
        {
          "glob": "favicon.ico",
          "input": "./",
          "output": "./"
        }
      ]
```

# Usage

In your component's module or app module. Import the following:

```
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { MonacoEditorLoaderModule, MonacoEditorLoaderService } from '@abc.xyz/angular-monaco-editor-loader';

import { AppComponent } from './app.component';
import { MonacoEditorComponent } from './monaco-editor/monaco-editor.component';

@NgModule({
  declarations: [
    AppComponent,
    MonacoEditorComponent
  ],
  imports: [
    BrowserModule,
    MonacoEditorLoaderModule <-- Insert this>
  ],
  bootstrap: [AppComponent]
})
export class AppModule { }

```
Just add `*loadMonacoEditor`, so with your custom component where you plan to create your own monaco component. Just add the following:

```
<monaco-editor *loadMonacoEditor></monaco-editor>
```

And in my custom component where I want to host `Monaco Editor` I just do the following like I expect the Monaco library to be loaded at this point:

```
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'monaco-editor',
  templateUrl: './monaco-editor.component.html',
  styleUrls: ['./monaco-editor.component.css']
})
export class MonacoEditorComponent implements OnInit {

  constructor() { }

  ngOnInit() {
    monaco.languages.typescript.javascriptDefaults.setDiagnosticsOptions({
      noSemanticValidation: true,
      noSyntaxValidation: false
    });

    // compiler options
    monaco.languages.typescript.javascriptDefaults.setCompilerOptions({
      target: monaco.languages.typescript.ScriptTarget.ES2016,
      allowNonTsExtensions: true
    });

    // extra libraries
    monaco.languages.typescript.javascriptDefaults.addExtraLib([
      'declare class Facts {',
      '    /**',
      '     * Returns the next fact',
      '     */',
      '    static next():string',
      '}',
    ].join('\n'), 'filename/facts.d.ts');

    var jsCode = [
      '"use strict";',
      '',
      "class Chuck {",
      "    greet() {",
      "        return Facts.next();",
      "    }",
      "}"
    ].join('\n');

    monaco.editor.create(document.getElementById("container"), {
      value: jsCode,
      language: "javascript"
    });
  }

}
```

And that's it! No `timeouts`! No `then`! It just goes with the correct flow in Angular!

# Running the demo app
Make sure you have **Angular CLI** installed!

1. Clone this repository
2. `cd demo`
3. `ng serve`

# Motivation

I did not want to clutter my component or code with `timeouts` or `then` to determine if Monaco has loaded! I also wanna utilize `ReactiveJS` when dealing with these kind of stuff.

Most of the code that was found [here](https://github.com/Microsoft/monaco-editor/issues/18) just wasn't the right timing when to check if Monaco is already loaded. 

Sometimes I see hacks from [Covalent](https://github.com/Teradata/covalent-code-editor/blob/develop/src/platform/code-editor/code-editor.component.ts) such as adding `timeouts`. So many timeouts everywhere!

```
        if (this._webview) {
            if (this._webview.send !== undefined) {
                // don't want to keep sending content if event came from IPC, infinite loop
                if (!this._fromEditor) {
                    this._webview.send('setEditorContent', value);
                }
                this.onEditorValueChange.emit(undefined);
                this.onChange.emit(undefined);
                this.propagateChange(this._value);
                this._fromEditor = false;
            } else {
                // Editor is not loaded yet, try again in half a second
                setTimeout(() => {
                    this.value = value;
                }, 500);
            }
        }
```

