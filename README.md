# Branding Visual Studio Code - Open Source

   <details>
          <summary>You can brand some of the UI elements in the Visual Studio Code - Open Source IDE with your corporate or product brand.</summary>

---

This means first adding brand-related files to the forked IDE repository, then building a container image of the branded IDE, and finally adding a `che-editor.yaml` file to the project repository.

Here are some examples of the UI elements in Visual Studio Code - Open Source that you can brand:

* Browser tab title and icon
* The icon for the empty editor area when no editor is open
* The Status Bar commands
* The Status Bar icon
* The Get Started page
* The tab icon for the Get Started page
* The application name in the *About* dialog


## Prerequisites

* Bash
* `{docker-cli}`

## Procedure

1. Fork or download the Git [repository](https://github.com/che-incubator/che-code/tree/main/) of Visual Studio Code - Open Source IDE for {prod}.

2. In the `/branding/` folder of the repository, create the `product.json` file, which maps custom branding resources.

	**TIP** In the `product.json` file, specify all paths relative the `/branding/` folder.

	*Example. `/branding/product.json`*

	The following example shows all of the properties that you can customize by using this file:

```
{
    "nameShort": "The application name for UI elements",
    "nameLong": "Red Hat OpenShift Dev Spaces with Microsoft Visual Studio Code - Open Source IDE",
    "icons": {
        "favicon": {
            "universal": "icons/favicon.ico"
        },
        "welcome": {
            "universal": "icons/dev-spaces.svg"
        },
        "statusBarItem": {
            "universal": "icons/dev-spaces.svg"
        },
        "letterpress": {
            "light": "icons/letterpress-light.svg",
            "dark": "icons/letterpress-light.svg"
        }
    },
    "remoteIndicatorCommands": {
        "openDocumentationCommand": "Dev Spaces: Open Documentation",
        "openDashboardCommand": "Dev Spaces: Open Dashboard",
        "stopWorkspaceCommand": "Dev Spaces: Stop Workspace"
    },
    "workbenchConfigFilePath": "workbench-config.json",
    "codiconCssFilePath": "css/codicon.css"
}
```

	`nameShort` is the application name for UI elements.

	`nameLong` is the application name that is used for the *Welcome* page, **About** dialog, and browser tab title.

	`favicon` is the icon for the browser tab title for all themes.

	`welcome` is the icon for the tab title of the **Get Started** page for all themes.

	`statusBarItem` is the icon for the bottom *Status Bar* for all themes. Define it as `codicon` in the `workbench-config.json` file and the `codicon` CSS styles.

	`letterpress` is the icon for the empty editor area when no editor is open. You can provide different icon files for `light` and `dark` themes.

	`remoteIndicatorCommands` is the names of commands provided by the [Eclipse Che Remote](https://github.com/che-incubator/che-code/blob/main/code/extensions/che-remote/package.nls.json) extension. Users can run these commands by clicking the *Status Bar*.

	`workbenchConfigFilePath` is the relative path to `workbench-config.json`, which is explained in one of the next steps.

	`codiconCssFilePath` is the relative path to `css/codicon.css`, which is explained in one of the next steps.

	**NOTE** The values defined in the `/branding/product.json` file override the [default values](https://github.com/che-incubator/che-code/blob/main/code/product.json).

3. Add the icon files, which you specified in the `product.json` file in the previous step, to the repository.

4. Create a `/branding/workbench-config.json` file with custom values.

	*Example. `/branding/workbench-config.json`*

```
{
	"windowIndicator": {
		"label": "$(eclipse-che) Eclipse Che",
		"tooltip": "Eclipse Che"
	},
	"configurationDefaults": {
		"workbench.colorTheme": "Dark",
		"workbench.colorCustomizations": {
			"statusBarItem.remoteBackground": "#FDB940",
			"statusBarItem.remoteForeground": "#525C86"
		}
	},
	"initialColorTheme": {
		"themeType": "dark",
		"colors": {
			"statusBarItem.remoteBackground": "#FDB940",
			"statusBarItem.remoteForeground": "#525C86"
		}
	}
}
```

5. Create a `/branding/css/codicon.css` file with custom values.

	*Example. `/branding/css/codicon.css`*

```
span.codicon.codicon-eclipse-che  {
	background-image: url(./che/che-icon.svg);
	width: 13px;
	height: 13px;
}
```

6. Run the `/branding/branding.sh` script.

```
$ ./branding/branding.sh
```

7. Build the container image from the `/che-code/` directory and push the image to a container registry:

```
$ {docker-cli} build -f build/dockerfiles/linux-musl.Dockerfile -t linux-musl-amd64 .

$ {docker-cli} build -f build/dockerfiles/linux-libc.Dockerfile -t linux-libc-amd64 .

$ export DOCKER_BUILDKIT=1

$ {docker-cli} build -f build/dockerfiles/assembly.Dockerfile -t vs-code-open-source .

$ {docker-cli} push <username>/vs-code-open-source:next
```

8. Create a `/.che/che-editor.yaml` file in the remote repository that you intend to clone into workspaces. This file must specify the container image of your customized Visual Studio Code - Open Source that is to be pulled for new workspaces.

	*Example. `/che-editor.yaml` for the branded Visual Studio Code - Open Source*

```
inline:
  schemaVersion: 2.1.0
  metadata:
    name: che-code
  commands:
    - id: init-container-command
      apply:
        component: che-code-injector
  events:
    preStart:
      - init-container-command
  components:
    - name: che-code-runtime-description
      container:
        image: quay.io/devfile/universal-developer-image:ubi8-latest
        command:
          - /checode/entrypoint-volume.sh
        volumeMounts:
          - name: checode
            path: /checode
        memoryLimit: 2Gi
        memoryRequest: 256Mi
        cpuLimit: 500m
        cpuRequest: 30m
        endpoints:
          - name: che-code
            attributes:
              type: main
              cookiesAuthEnabled: true
              discoverable: false
              urlRewriteSupported: true
            targetPort: 3100
            exposure: public
            secure: false
            protocol: https
          - name: code-redirect-1
            attributes:
              discoverable: false
              urlRewriteSupported: true
            targetPort: 13131
            exposure: public
            protocol: http
          - name: code-redirect-2
            attributes:
              discoverable: false
              urlRewriteSupported: true
            targetPort: 13132
            exposure: public
            protocol: http
          - name: code-redirect-3
            attributes:
              discoverable: false
              urlRewriteSupported: true
            targetPort: 13133
            exposure: public
            protocol: http
      attributes:
        app.kubernetes.io/component: che-code-runtime
        app.kubernetes.io/part-of: che-code.eclipse.org
    - name: checode
      volume: {}
    - name: che-code-injector
      container:
        image: quay.io/username/vs-code-open-source:next <1>
        command: ["/entrypoint-init-container.sh"]
        volumeMounts:
          - name: checode
            path: /checode
        memoryLimit: 128Mi
        memoryRequest: 32Mi
        cpuLimit: 500m
        cpuRequest: 30m
```

	**NOTE** In this example, `quay.io/username/vs-code-open-source:next` specifies the container image of a branded Visual Studio Code - Open Source that will be pulled at workspace creation.


## Verification

1. Start a new workspace with a clone of the project repository that contains the `che-editor.yaml` file.

2. Check that the configured UI elements are correctly branded in Visual Studio Code - Open Source in the workspace.

---

</details>
