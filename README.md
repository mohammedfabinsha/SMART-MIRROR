


**SmartMirror** is an open source modular smart mirror platform. With a growing list of installable modules, the **SmartMirror** allows you to convert your hallway or bathroom mirror into your personal assistant. **SmartMirror** is built by the creator of [the original SmartMirror] with the incredible help of a great circle of collegues and mentor prof.SHEENA.S .

SmartMirror focuses on a modular plugin system and uses [Electron](http://electron.atom.io/) as an application wrapper. So no more web server or browser installs necessary!

## Table Of Contents

- [Installation](#installation)
  - [Raspberry Pi](#raspberry-pi)
  - [General](#general)
  - [Server Only](#server-only)
  - [Client Only](#client-only)
  - [Docker](#docker)
- [Configuration](#configuration)
- [Modules](#modules)
- [Updating](#updating)
- [Known Issues](#known-issues)
- [Community](#community)
- [Contributing Guidelines](#contributing-guidelines)
- [Manifesto](#manifesto)

## Installation

### Raspberry Pi

#### Automatic Installation (Raspberry Pi only!)

*Electron*, the app wrapper around SmartMirror, only supports the Raspberry Pi 2/3. The Raspberry Pi 0/1 is currently **not** supported. If you want to run this on a Raspberry Pi 1, use the [server only](#server-only) feature and setup a fullscreen browser yourself. (Yes, people have managed to run SM also on a Pi0, so if you insist, search in the forums.)

Note that you will need to install the latest full version of Raspbian, **don't use the Lite version**.

Execute the following command on your Raspberry Pi to install SmartMirror:

```bash
bash -c "$(curl -sL https://raw.githubusercontent.com/ANANGH/SMART-MIRROR/master/installers/raspberry.sh)"
```

#### Manual Installation

1. Download and install the latest *Node.js* version:
- `curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash -`
- `sudo apt install -y nodejs`
2. Clone the repository and check out the master branch: `git clone https://github.com/ANANGH/SMART-MIRROR`
3. Enter the repository: `cd SMART-MIRROR/`
4. Install and run the app with: `npm install && npm start` \
   For **Server Only** use: `npm install && node serveronly` .


**:warning: Important!**

- **The installation step for `npm install` will take a very long time**, often with little or no terminal response! \
  For the RPi3 this is **~10** minutes and for the Rpi2 **~25** minutes. \
  Do not interrupt or you risk getting a :broken_heart: by Raspberry Jam.


Also note that:

- `npm start` does **not** work via SSH. But you can use `DISPLAY=:0 nohup npm start &` instead. \
  This starts the mirror on the remote display.
- If you want to debug on Raspberry Pi you can use `npm start dev` which will start MM with *Dev Tools* enabled.
- To access toolbar menu when in mirror mode, hit `ALT` key.
- To toggle the (web) `Developer Tools` from mirror mode, use `CTRL-SHIFT-I` or `ALT` and select `View`.


### Server Only

In some cases, you want to start the application without an actual app window. In this case, you can start SmartMirror in server only mode by manually running `node serveronly` or using Docker. This will start the server, after which you can open the application in your browser of choice. Detailed description below.

**Important:** Make sure that you whitelist the interface/ip (`ipWhitelist`) in the server config where you want the client to connect to, otherwise it will not be allowed to connect to the server. You also need to set the local host `address` field to `0.0.0.0` in order for the RPi to listen on all interfaces and not only `localhost` (default).

```javascript
var config = {
	address: "0.0.0.0",	// default is "localhost"
	port: 8080,		// default
	ipWhitelist: ["127.0.0.1", "::ffff:127.0.0.1", "::1", "::ffff:172.17.0.1"], // default -- need to add your IP here
	...
};
```


### Client Only

This is when you already have a server running remotely and want your RPi to connect as a standalone client to this instance, to show the MM from the server. Then from your RPi, you run it with: `node clientonly --address 192.168.1.5 --port 8080`. (Specify the ip address and port number of the server)


### Docker

SmartMirror in server only mode can be deployed using [Docker](https://docker.com). After a successful [Docker installation](https://docs.docker.com/engine/installation/) you just need to execute the following command in the shell:

```bash
docker run  -d \
	--publish 80:8080 \
	--restart always \
	--volume ~/SMART-MIRROR/config:/opt/SMART-MIRROR/config \
	--volume ~/SMART-MIRROR/modules:/opt/SMART-MIRROR/modules \
	--name SMART-MIRROR \
	bastilimbach/docker-SMART-MIRROR
```
To get more information about the available Dockerfile versions and configurations head over to the respective [GitHub repository](https://github.com/bastilimbach/docker-MagicMirror).


## Configuration

### Raspberry Specific

The following wiki links are helpful for the initial configuration of your SmartMirror operating system:
- [Configuring the Raspberry Pi](https://github.com/MichMich/MagicMirror/wiki/Configuring-the-Raspberry-Pi)
- [Auto Starting SmartMirror](https://github.com/MichMich/MagicMirror/wiki/Auto-Starting-MagicMirror)


### General

1. Copy `/home/pi/SMART-MIRROR/config/config.js.sample` to `/home/pi/SMART-MIRROR/config/config.js`. \
   **Note:** If you used the installer script. This step is already done for you.

2. Modify your required settings. \
   Note: You'll can check your configuration running `npm run config:check` in `/home/pi/SMART-MIRROR`.


The following properties can be configured:

| **Option** | **Description** |
| --- | --- |
| `port` | The port on which the SmartMirror server will run on. The default value is `8080`. |
| `address` | The *interface* ip address on which to accept connections. The default is `localhost`, which would prevent exposing the built-in webserver to machines on the local network. To expose it to other machines, use: `0.0.0.0`. |
| `ipWhitelist` | The list of IPs from which you are allowed to access the SmartMirror. The default value is `["127.0.0.1", "::ffff:127.0.0.1", "::1"]`, which is from `localhost` only. Add your IP when needed. You can also specify IP ranges with subnet masks (`["127.0.0.1", "127.0.0.1/24"]`) or directly with (`["127.0.0.1", ["192.168.0.1", "192.168.0.100"]]`). Set `[]` to allow all IP addresses. For more information see: [follow post ipWhitelist HowTo](https://forum.magicmirror.builders/topic/1326/ipwhitelist-howto) |
| `zoom` | This allows to scale the mirror contents with a given zoom factor. The default value is `1.0`|
| `language` | The language of the interface. (Note: Not all elements will be localized.) Possible values are `en`, `nl`, `ru`, `fr`, etc., but the default value is `en`. |
| `timeFormat` | The form of time notation that will be used. Possible values are `12` or `24`. The default is `24`. |
| `units` | The units that will be used in the default weather modules. Possible values are `metric` or `imperial`. The default is `metric`. |
| `modules` | An array of active modules. **The array must contain objects. See the next table below for more information.** |
| `electronOptions` | An optional array of Electron (browser) options. This allows configuration of e.g. the browser screen size and position (example: `electronOptions: { fullscreen: false, width: 800, height: 600 }`). Kiosk mode can be enabled by setting `kiosk: true`, `autoHideMenuBar: false` and `fullscreen: false`. More options can be found [here](https://github.com/electron/electron/blob/master/docs/api/browser-window.md). |
| `customCss` | The path of the `custom.css` stylesheet. The default is `css/custom.css`. |

Module configuration:

| **Option** | **Description** |
| --- | --- |
| `module` | The name of the module. This can also contain the subfolder. Valid examples include `clock`, `default/calendar` and `custommodules/mymodule`. |
| `position` | The location of the module in which the module will be loaded. Possible values are `top_bar`, `top_left`, `top_center`, `top_right`, `upper_third`, `middle_center`, `lower_third`, `bottom_left`, `bottom_center`, `bottom_right`, `bottom_bar`, `fullscreen_above`, and `fullscreen_below`. This field is optional but most modules require this field to set. Check the documentation of the module for more information. Multiple modules with the same position will be ordered based on the order in the configuration file. |
| `classes` | Additional classes which are passed to the module. The field is optional. |
| `header` | To display a header text above the module, add the header property. This field is optional. |
| `disabled` | Set disabled to `true` to skip creating the module. This field is optional. |
| `config` | An object with the module configuration properties. Check the documentation of the module for more information. This field is optional, unless the module requires extra configuration. |

## Modules

The following modules are installed by default.

- [**Clock**](modules/default/clock)
- [**Calendar**](modules/default/calendar)
- [**Current Weather**](modules/default/currentweather)
- [**Weather Forecast**](modules/default/weatherforecast)
- [**News Feed**](modules/default/newsfeed)
- [**Compliments**](modules/default/compliments)
- [**Hello World**](modules/default/helloworld)
- [**Alert**](modules/default/alert)

For more available modules, check out out the wiki page [SmartMirror 3rd Party Modules](https://github.com/ANANGH/SMART-MIRROR/wiki/3rd-party-modules). If you want to build your own modules, check out the [SmartMirror Module Development Documentation](modules) and don't forget to add it to the wiki and the [forum](https://forum.magicmirror.builders/category/7/showcase)!


## Updating

If you want to update your SmartMirror to the latest version, use your terminal to go to your Magic Mirror folder and type the following command:

```bash
git pull && npm install
```

If you changed nothing more than the config or the modules, this should work without any problems.
Type `git status` to see your changes, if there are any, you can reset them with `git reset --hard`. After that, git pull should be possible.


## Contributing Guidelines

Contributions of all kinds are welcome, not only in the form of code but also with regards bug reports and documentation.
Please keep the following in mind:

- **Bug Reports**:  Make sure you're running the latest version. If the issue(s) still persist: please open a clearly documented issue with a clear title.
- **Minor Bug Fixes**: Please send a pull request with a clear explanation of the issue or a link to the issue it solves.
- **Major Bug Fixes**: please discuss your approach in an GitHub issue before you start to alter a big part of the code.
- **New Features**: please please discuss in a GitHub issue before you start to alter a big part of the code. Without discussion upfront, the pull request will not be accepted / merged.

Thanks for your help in making SmartMirror better!

## Manifesto

A real Manifesto is still to be written. Till then, Michael's response on [one of the repository issues]https://github.com/ANANGH/SMART-MIRROR/issues/1174) gives a great summary:

> "... I started this project as an ultimate starter project for Raspberry Pi enthusiasts. As a matter of fact, for most of the contributors, the SmartMirror project is the first open source project they ever contributed to. This is one of the reasons why the SmartMirror project is featured in several RasPi magazines.
>
>The project has a lot of opportunities for improvement. We could use a powerful framework like Vue to ramp up the development speed. We could use SASS for better/easier css implementations. We could make it an NPM installable package. And as you say, we could bundle it up. The big downside of of of these changes is that it over complicates things: a user no longer will be able to open just one file and make a small modification and see how it works out.
>
>Of course, a bundled version can be complimentary to the regular un-bundled version. And I'm sure a lot of (new) users will opt for the bundled version. But this means those users won't be motivated to take a peek under the hood. They will just remain 'users'. They won't become contributors, and worse: they won't be motivated to take their first steps in software development.
>
>And to be honest: motivating curious users to step out of their comfort zone and take those first steps is what drives me in this project. Therefor my ultimate goal is this project is to keep it as accessible as possible."
>
