<!--
SPDX-FileCopyrightText: NOI Techpark <digital@noi.bz.it>

SPDX-License-Identifier: CC0-1.0
-->

# Events Today

[![REUSE Compliance](https://github.com/noi-techpark/webcomp-events-today/actions/workflows/reuse.yml/badge.svg)](https://github.com/noi-techpark/odh-docs/wiki/REUSE#badges)

Events Today is a web component that displays a list of events that are happening at NOI Techpark. It fetches data from the [EventShort API](https://tourism.opendatahub.com/swagger/index.html#/EventShort)  and displays the events in a simple, easy-to-read format.<br>


**Table of Contents**
- [Usage](#usage)
- [License](#license)
- [Support](#support)

## Usage
To use the "Events Today" web component, you need to include the following code in your HTML file: 
```html
<events-today></events-today>
```
### General attributes

#### <b>room</b>
The room attribute allows you to select events based on the specific room or location where the event will be held. The room attribute can be used as a filter to narrow down the search results and show only events that are taking place in a particular room or location.  
Default value is empty, so all rooms are shown.

```
<events-today room="Seminar 1"></events-today>
```

## Getting started

These instructions will get you a copy of the project up and running
on your local machine for development and testing purposes.

### Prerequisites

To build the project, the following prerequisites must be met:

- Node 16.x / NPM 9.x

For a ready to use Docker environment with all prerequisites already installed
and prepared, you can check out the [Docker environment](#docker-environment)
section.

### Source code

Get a copy of the repository:

```bash
git clone git@github.com:noi-techpark/webcomp-events-today.git
```

Change directory:

```bash
cd webcomp-events-today/
```

### Dependencies

Download all dependencies:

```bash
npm install
```

### Build

Build and start the project:

```bash
npm start
```

You can the open the created `dist/demo.html` file in your browser.

## Deployment

To create the distributable files, execute the following command:

```bash
npm run build
```

## Information

### Support

For support, please contact [help@opendatahub.com](mailto:help@opendatahub.com).

### Contributing

If you'd like to contribute, please follow the Contributor Guidelines that can be found at [https://github.com/noi-techpark/odh-docs/wiki/Contributor-Guidelines%3A-Getting-started](https://github.com/noi-techpark/odh-docs/wiki/Contributor-Guidelines%3A-Getting-started).

### Documentation

More documentation can be found at [https://opendatahub.readthedocs.io/en/latest/index.html](https://opendatahub.readthedocs.io/en/latest/index.html).

### Boilerplate

The project uses this boilerplate: [https://github.com/noi-techpark/webcomp-boilerplate](https://github.com/noi-techpark/webcomp-boilerplate).

### License

The code in this project is licensed under the GNU AFFERO GENERAL PUBLIC LICENSE Version 3 license. See the [LICENSE.md](LICENSE.md) file for more information.

## REUSE

This project is [REUSE](https://reuse.software) compliant, more information about the usage of REUSE in NOI Techpark repositories can be found [here](https://github.com/noi-techpark/odh-docs/wiki/Guidelines-for-developers-and-licenses#guidelines-for-contributors-and-new-developers).

Since the CI for this project checks for REUSE compliance you might find it useful to use a pre-commit hook checking for REUSE compliance locally. The [pre-commit-config](.pre-commit-config.yaml) file in the repository root is already configured to check for REUSE compliance with help of the [pre-commit](https://pre-commit.com) tool.

Install the tool by running:
```bash
pip install pre-commit
```
Then install the pre-commit hook via the config file by running:
```bash
pre-commit install
```

