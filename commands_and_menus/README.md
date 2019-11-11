# Commands and Menus: Extending the main app

* [Jupyterlab Commands](#jupyterlab-commands)
* [Adding new Menu tabs and items](#adding-new-menu-tabs-and-items)

For the next extension you can either copy the last folder to a new one or
simply continue modifying it. In case that you want to have a new extension,
open the file `package.json` and modify the package name, e.g. into
`commands_and_menus`. The same name changes needs to be done in
`src/index.ts`.

## Jupyterlab Commands

Start it with `jupyter lab --watch`. In this extension, we are going to add a
command to the application command registry and expose it to the user in the
command palette.  The command palette can be seen when clicking on _Commands_
on the left hand side of Jupyterlab. The command palette can be seen as a list
of actions that can be executed by JupyterLab. (see screenshot below).

![Jupyter Command Registry](_images/command_registry.png)

Extensions can provide a bunch of functions to the JupyterLab command registry
and then expose them to the user through the command palette or through a menu
item.

Two types play a role in this: the `CommandRegistry` type ([documentation](http://phosphorjs.github.io/phosphor/api/commands/classes/commandregistry.html))
and the command palette interface `ICommandPalette` that is imported with:

```typescript
import {
  ICommandPalette
} from '@JupyterLab/apputils';
```

To see how we access the applications command registry and command palette
open the file `src/index.ts`.

```typescript
const extension: JupyterLabPlugin<void> = {
    id: 'commands_and_menus',
    autoStart: true,
    requires: [ICommandPalette],
    activate: (
        app: JupyterLab,
        palette: ICommandPalette) =>
    {
        const { commands } = app;

        let command = 'labtutorial';
        let category = 'Tutorial';

        commands.addCommand(command, {
            label: 'New Labtutorial',
            caption: 'Open the Labtutorial',
            execute: (args) => {console.log('Hey')}});

        palette.addItem({command, category});
    }
};

export default extension;
```

The CommandRegistry is an attribute of the main JupyterLab application
(variable `app` in the previous section). It has an `addCommand` method that
adds our own function.
The ICommandPalette
([documentation](https://JupyterLab.github.io/JupyterLab/interfaces/_apputils_src_commandpalette_.icommandpalette.html))
is passed to the `activate` function as an argument (variable `palette`) in
addition to the JupyterLab application (variable `app`). We specify with the
property `requires: [ICommandPalette],` which additional arguments we want to
inject into the `activate` function in the JupyterLabPlugin. ICommandPalette
provides the method `addItem` that links a palette entry to a command in the
command registry. Our new plugin code then becomes:

When this extension is build (and linked if necessary), JupyterLab looks like
this:

![New Command](_images/new_command.png)

## Adding new Menu tabs and items

Adding new menu items works in a similar way. The IMainMenu interface can be
passed as a new argument two the activate function, but first it has to be
imported. The Menu class is imported from the phosphor library on top of which
JupyterLab is built, and that will be frequently encountered when developing
JupyterLab extensions:

```typescript
import {
  IMainMenu
} from '@JupyterLab/mainmenu';

import {
  Menu
} from '@phosphor/widgets';
```

We add the IMainMenu in the `requires:` property such that it is injected into
the `activate` function. The Extension is then changed to:

```typescript
const extension: JupyterLabPlugin<void> = {
    id: 'commands_and_menus',
    autoStart: true,
    requires: [ICommandPalette, IMainMenu],
    activate: (
        app: JupyterLab,
        palette: ICommandPalette,
        mainMenu: IMainMenu) =>
    {
        const { commands } = app;
        let command = 'labtutorial';
        let category = 'Tutorial';
        commands.addCommand(command, {
            label: 'New Labtutorial',
            caption: 'Open the Labtutorial',
            execute: (args) => {console.log('Hey')}});
        palette.addItem({command, category});

        let tutorialMenu: Menu = new Menu({commands});

        tutorialMenu.title.label = 'Tutorial';
        mainMenu.addMenu(tutorialMenu, {rank: 80});
        tutorialMenu.addItem({ command });
    }
};

export default extension;
```

In this extension, we have added the dependencies _JupyterLab/mainmenu_ and
_phosphor/widgets_. Before it builds, this dependencies have to be added to the
`package.json` file:

```json
  "dependencies": {
    "@JupyterLab/application": "^0.16.0",
    "@JupyterLab/mainmenu": "*",
    "@phosphor/widgets": "*"
  }
```

we can then do

```
npm install
npm run build
```

to rebuild the application. After a browser refresh, the JupyterLab website
should now show:

![New Menu](_images/new_menu.png)

[ps:

for the build to run, the `tsconfig.json` file might have to be updated to:
```
{
  "compilerOptions": {
    "declaration": true,
    "noImplicitAny": true,
    "noEmitOnError": true,
    "noUnusedLocals": true,
    "module": "commonjs",
    "moduleResolution": "node",
    "target": "ES6",
    "outDir": "./lib",
    "lib": ["ES6", "ES2015.Promise", "DOM"],
    "types": []
  },
  "include": ["src/*"]
}
```
]

[Click here for the final extension commands_and_menus](commands_and_menus)