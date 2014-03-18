Angular UI Tree
======================

[![Build Status](https://travis-ci.org/JimLiu/angular-ui-tree.png?branch=master)](https://travis-ci.org/JimLiu/angular-ui-tree)

Angular UI Tree is an AngularJS UI component that can sort nested lists, provides drag & drop support and doesn't depend on jQuery.

## Features

- Uses the native AngularJS scope for data binding
- Sorted and move items through the entire tree
- Prevent elements from accepting child nodes

## Supported browsers

The Angular UI Tree is tested with the following browsers:

- Chrome (stable)
- Firefox
- IE 8, 9 and 10

For IE8 support, make sure you do the following:

- include an [ES5 shim](https://github.com/es-shims/es5-shim)
- make your [AngularJS application compatible with Internet Explorer](http://docs.angularjs.org/guide/ie)
- use [jQuery 1.x](http://jquery.com/browser-support/)

## Demo
Watch the Tree component in action on the [demo page](http://jimliu.github.io/angular-ui-tree/).

## Requirements

- Angularjs

## Usage

### Download
- Using [bower](http://bower.io/) to install it. `bower install angular-ui-tree`
- [Download](https://github.com/JimLiu/angular-ui-tree/archive/master.zip) from github.

### Load CSS
Load the css file: `angular-ui-tree.min.css` in your application:
```html
<link rel="stylesheet" href="bower_components/angular-ui-tree/angular-ui-tree.min.css">
```


### Load Script
Load the script file: `angular-ui-tree.js` or `angular-ui-tree.min.js` in your application:

```html
<script type="text/javascript" src="bower_components/angular-ui-tree/angular-ui-tree.js"></script>
```

### Code
Add the sortable module as a dependency to your application module:

```js
var myAppModule = angular.module('MyApp', ['ui.tree'])
```

Injecting `ui.tree`, `ui-tree-nodes` and `ui-tree-node` to your html.

#### HTML View or Templates
```html
<div ui-tree>
  <ol ui-tree-nodes="" ng-model="list">
    <li ng-repeat="item in list" ui-tree-node>
      <div class="angular-ui-tree-handle">
        {{item.title}}
      </div>
      <ol ui-tree-nodes="" ng-model="item.items">
        <li ng-repeat="subItem in item.items" ui-tree-node>
          <div class="angular-ui-tree-handle">
            {{subItem.title}}
          </div>
        </li>
      </ol>
    </li>      
  </ol> 
</div>
```  
**Developing Notes:**
- Adding `ui-tree` to your root element of the tree.
- Adding `ui-tree-nodes` to the elements which contain the nodes. `ng-model` is required, and it should be an array, so that the directive knows which model to bind and update.
- Adding `ui-tree-node` to your node element, it always follows the `ng-repeat` attribute.
- All `ui-tree-nodes`, `ng-model`, `ui-tree-node` are necessary. And they can be nested.
- If you changed the datasource bound, sometimes you have to call [`$scope.$apply()`](http://docs.angularjs.org/api/ng/type/$rootScope.Scope#$apply) to refresh the view, otherwise you will get an error `Cannot read property '0' of undefined` ([Issue #32](https://github.com/JimLiu/angular-ui-tree/issues/32)).

#### Unlimited nesting HTML View or Templates Example

```html
<!-- Nested node template -->
<script type="text/ng-template" id="nodes_renderer.html">
  <div class="angular-ui-tree-handle">
    {{node.title}}
  </div>
  <ol ui-tree-nodes="" ng-model="node.nodes">
    <li ng-repeat="node in node.nodes" ui-tree-node ng-include="'nodes_renderer.html'">
    </li>
  </ol>
</script>
<ol ui-tree-nodes="" ng-model="data" id="tree-root">
  <li ng-repeat="node in data" ui-tree-node ng-include="'nodes_renderer.html'"></li>
</ol>
```

## Structure of angular-ui-tree

    ui-tree                             --> Root of tree
        ui-tree-nodes                   --> Container of nodes
            ui-tree-node                --> One of the node of a tree
                ui-tree-nodes           --> Container of child-nodes
                    ui-tree-node        --> Child node
                    ui-tree-node        --> Child node
            ui-tree-node                --> Another node

## API

### ui-tree
`ui-tree` is the root scope for a tree

#### Attributes
##### data-drag-enabled
Turn on dragging and dropping of nodes.
- `true` (default): allow drag and drop
- `false`: turn off drag and drop
Example: turn on/off drag and drop.
```html
<div ui-tree data-drag-enabled="tree.enabled">
    
</div>
```

### ui-tree-nodes
`ui-tree-nodes` is the container of nodes. Every `ui-tree-node` should have a `ui-tree-nodes` as it's container, a `ui-tree-nodes` can have multiple child nodes.

#### Attributes
##### data-nodrop
Turn off drop of nodes. It can be overwritten by [$callbacks.accept](#accept).
Example: turn off drop.
```html
<ol ui-tree-nodes ng-model="nodes" data-nodrop>
  <li ng-repeat="node in nodes" ui-tree-node>{{node.title}}</li>      
</ol> 
```

#### Properties of scope
##### $element (type: AngularElement)
The html element which bind with the `ui-tree-nodes` scope.

##### $modelValue (type: Object)
The data which bind with the scope.

##### $nodes (type: Array)
All it's child nodes. The type of child node is scope of `ui-tree-node`.

##### $nodeScope (type: Scope of ui-tree-node)
The scope of node which current `ui-tree-nodes` belongs to.
For example:

    ui-tree-nodes                       --> nodes 1
            ui-tree-node                --> node 1.1
                ui-tree-nodes           --> nodes 1.1
                    ui-tree-node        --> node 1.1.1
                    ui-tree-node        --> node 1.1.2
            ui-tree-node                --> node 1.2

The property `$nodeScope of` `nodes 1.1` is `node 1.1`. The property `$nodes` of `nodes 1.1` is [`node 1.1.1`, `node 1.1.2`]

##### $callbacks (type: Object)
`$callbacks` is a very important property for `angular-ui-tree`. When some special events trigger, the functions in `$callbacks` are called. The Callbacks can be passed through the directive. 
Example: 
```js
myAppModule.controller('MyController', function($scope) {
  $scope.treeNodesOptions = {
    accept: function(sourceNodeScope, destNodesScope, destIndex) {
      return true;
    },
  };
});
```
```html
<ol ui-tree-nodes="treeNodesOptions" ng-model="nodes">
  <li ng-repeat="node in nodes" ui-tree-node>{{node.title}}</li>      
</ol> 
```

#### Methods of scope
##### collapseAll()
Collapse all it's child nodes.

##### expandAll()
Expand all it's child nodes.

#### Methods in $callbacks
##### <a name="accept"></a>accept
Check if the current dragging node can be dropped in the `ui-tree-nodes`.

**Parameters:**
- `sourceNodeScope`: The scope of source node which is dragging.
- `destNodesScope`: The scope of `ui-tree-nodes` which you want to drop in.
- `destIndex`: The position you want to drop in.

**Return**
If the nodes accept the current dragging node.
- `true` Allow it to drop.
- `false` Not allow.

##### <a name="dragStart"></a>dragStart
The `dragStart` function is called when the user starts to drag the node.
**<a name="dragStartParams"></a>Parameters:**
- `sourceNodeScope`: The current dragging node.
- `elements`: The dragging relative elements.
    + `placeholder`: The placeholder element.
    + `drag`: The dragging element.
- `pos`: Position object.

##### dragMove
The `dragMove` function is called when the user moves the node.

**Parameters:**
Same as [Parameters](#dragStartParams) of dragStart.

##### dragStop
The `dragStop` function is called when the user stop dragging the node.

**Parameters:**
Same as [Parameters](#dragStartParams) of dragStart.


### ui-tree-node
A node of a tree. Every `ui-tree-node` should have a `ui-tree-nodes` as it's container.

#### Attributes
##### data-nodrag
Turn off drag of node.
Example: turn off drag.
```html
<ol ui-tree-nodes ng-model="nodes">
  <li ng-repeat="node in nodes" ui-tree-node data-nodrag>{{node.title}}</li>
</ol>
```

#### Properties of scope
##### $element (type: AngularElement)
The html element which bind with the `ui-tree-nodes` scope.

##### $modelValue (type: Object)
The data which bind with the scope.

##### collapsed (type: Bool)
If the node is collapsed

- `true`: Current node is collapsed;
- `false`: Current node is expanded.

##### $parentNodeScope (type: Scope of ui-tree-node)
The scope of parent node.

##### $childNodesScope (type: Scope of ui-tree-nodes)
The scope of it's `ui-tree-nodes`.

##### $parentNodesScope (type: Scope of ui-tree-nodes)
The scope of it's parent `ui-tree-nodes`.

For example:

    ui-tree-nodes                       --> nodes 1
            ui-tree-node                --> node 1.1
                ui-tree-nodes           --> nodes 1.1
                    ui-tree-node        --> node 1.1.1
                    ui-tree-node        --> node 1.1.2
            ui-tree-node                --> node 1.2

- `node 1.1.1`.`$parentNodeScope` is `node 1.1`.
- `node 1.1`.`$childNodesScope` is `nodes 1.1`.
- `node 1.1`.`$parentNodesScope` is `nodes 1`.

#### Methods of scope
##### collapse()
Collapse current node.

##### expand()
Expand current node.

##### toggle()
Toggle current node.

##### remove()
Remove current node.

## NgModules Link

[Give us a like on ngmodules](http://ngmodules.org/modules/Angular-NestedSortable)

## Development environment setup
#### Prerequisites

* [Node Package Manager](https://npmjs.org/) (NPM)
* [Git](http://git-scm.com/)

#### Dependencies

* [Grunt](http://gruntjs.com/) (task automation)
* [Bower](http://bower.io/) (package management)

#### Installation
Run the commands below in the project root directory.

#####1. Install Grunt and Bower

    $ sudo npm install -g grunt-cli bower
    
#####2. Install project dependencies

    $ npm install
    $ bower install

## Useful commands

####Running a Local Development Web Server
To debug code and run end-to-end tests, it is often useful to have a local HTTP server. For this purpose, we have made available a local web server based on Node.js.

To start the web server, run:

    $ grunt webserver

To access the local server, enter the following URL into your web browser:

    http://localhost:8080/demo/

By default, it serves the contents of the demo project.


####Building angular-ui-tree
To build angular-ui-tree, you use the following command.

    $ grunt build

This will generate non-minified and minified JavaScript files in the `dist` directory.

####Run tests
You can run the tests once or continuous.

    $ grunt test
    $ grunt test:continuous
