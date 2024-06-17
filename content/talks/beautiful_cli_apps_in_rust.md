---
title: 'Beautiful CLI Apps in Rust'
date: "2022-10-18T14:45:00"
categories: ["workshop"]
tags: ["opensuse", "packaging"]
video: https://talks.codemotion.com/wannabe-speaker-1
---
background-image: url(/img/beautiful_cli_apps_in_rust/codemotion_first_page.png)
class: left, middle
name: start

# Beautiful CLI Applications</br> in Rust ![:i](fab fa-rust)

### Danilo Spinella

---
background-image: url(/img/beautiful_cli_apps_in_rust/codemotion_page.png)
class: left, middle

# You can follow at

https://shorturl.at/CDFIX

![](/img/beautiful_cli_apps_in_rust/qr.jpg)

---
background-image: url(/img/beautiful_cli_apps_in_rust/codemotion_page.png)
class: left, middle

.column[## Danilo Spinella

_Software Engineer in Packaging_

**SUSE** ![:i](fab fa-suse)]

.column[![:resize 200](/img/profile.png)]

---
background-image: url(/img/beautiful_cli_apps_in_rust/codemotion_page.png)
class: left, middle

## ![:i](fas fa-list) Agenda

Design of the example application

Implementation

---
background-image: url(/img/beautiful_cli_apps_in_rust/codemotion_page.png)
class: center, middle

# Design of the example application

---
background-image: url(/img/beautiful_cli_apps_in_rust/codemotion_page.png)
class: left, middle

## Weather CLI app (1/2)

Fetches the current weather for a city and gives us the result

---
class: center, middle

![:resize 850](/img/beautiful_cli_apps_in_rust/example_cli.png)

---
background-image: url(/img/beautiful_cli_apps_in_rust/codemotion_page.png)
class: left, middle

## Why Rust ![:i](fa fa-question)

Ease of development

Robustness

Solid error handling

Complete application in 73 LOC

---
background-image: url(/img/beautiful_cli_apps_in_rust/codemotion_page.png)
class: left, middle

## OpenWeather API

Provides free weather forecasts

It requires a valid API key to work

---
background-image: url(/img/beautiful_cli_apps_in_rust/codemotion_page.png)
class: left, middle

## Weather CLI app (2/2)

Parse the name of the city from the command line

Read the OpenWeather API key from a config file

Fetch and parse the data

Print it to the user in a nice way

---
background-image: url(/img/beautiful_cli_apps_in_rust/codemotion_page.png)
class: middle, center

# Implementation

---
background-image: url(/img/beautiful_cli_apps_in_rust/codemotion_page.png)
class: left, middle

## Crates

Parse the name of the city from the command line ![:i](fas fa-arrow-right) **clap**

Read the OpenWeather API key from a config file ![:i](fas fa-arrow-right) **directories**, **serde** and **yaml**

Fetch and parse the data ![:i](fas fa-arrow-right)  **reqwest** and **serde-json**

Print it to the user in a nice way ![:i](fas fa-arrow-right) **colored**

---
background-image: url(/img/beautiful_cli_apps_in_rust/codemotion_page.png)
class: left, middle

## Error handling

Function that can fail almost always return a `Result` wrapper over the returned value

![:resize 850](/img/beautiful_cli_apps_in_rust/result_code.svg)

`unwrap()` -> returns the inner value and panic on error

`expect(err_msg)` -> returns the inner value and panic on error with a specified message

---
background-image: url(/img/beautiful_cli_apps_in_rust/codemotion_page.png)
class: left, middle

## Clap

Powerful and flexible command line argument library

![:resize 850](/img/beautiful_cli_apps_in_rust/clap_code.svg)

---
background-image: url(/img/beautiful_cli_apps_in_rust/codemotion_page.png)
class: left, middle

## directories

Provides standard locations of directories for config, cache and other data on all platforms

![:resize 850](/img/beautiful_cli_apps_in_rust/directories_code.svg)

---
background-image: url(/img/beautiful_cli_apps_in_rust/codemotion_page.png)
class: left, middle

## serde (1/2)

De-facto standard serialization and deserialization library

![:resize 850](/img/beautiful_cli_apps_in_rust/serde_code_1.svg)

---
background-image: url(/img/beautiful_cli_apps_in_rust/codemotion_page.png)
class: left, middle

## yaml

Implement YAML data format for serde

![:resize 850](/img/beautiful_cli_apps_in_rust/serde_yaml_code.svg)

---
background-image: url(/img/beautiful_cli_apps_in_rust/codemotion_page.png)
class: left, middle

## reqwest (1/2)

Provides a convenient, higher-level HTTP Client

![:resize 850](/img/beautiful_cli_apps_in_rust/reqwest_code_1.svg)

---
background-image: url(/img/beautiful_cli_apps_in_rust/codemotion_page.png)
class: left, middle

## reqwest (2/2)

![:resize 850](/img/beautiful_cli_apps_in_rust/reqwest_code_2.svg)

---
background-image: url(/img/beautiful_cli_apps_in_rust/codemotion_page.png)
class: left, middle

## serde (2/2)

![:resize 850](/img/beautiful_cli_apps_in_rust/serde_json_2.svg)

---
background-image: url(/img/beautiful_cli_apps_in_rust/codemotion_page.png)
class: left, middle

## serde-json

Implements JSON data format for serde

![:resize 850](/img/beautiful_cli_apps_in_rust/serde_json_1.svg)

---
background-image: url(/img/beautiful_cli_apps_in_rust/codemotion_page.png)
class: left, middle

## colored

![:resize 850](/img/beautiful_cli_apps_in_rust/colored_code.svg)

---
background-image: url(/img/beautiful_cli_apps_in_rust/codemotion_page.png)
class: left, middle

## You can find the result here

https://github.com/danyspin97/weather-cli-app-codemotion

---
background-image: url(/img/beautiful_cli_apps_in_rust/codemotion_page.png)
class: left, middle

## Thank you for your attention!

# Questions ![:i](fas fa-question)

---
background-image: url(/img/beautiful_cli_apps_in_rust/codemotion_page.png)
class: left, middle

# Contacts

![:i](fas fa-globe) https://danyspin97.org

![:i](fas fa-envelope) [oss@danyspin97.org](mailto://oss@danyspin97.org)

![:i](fab fa-github) [danyspin97](https://github.com/danyspin97.org)

---
template: start

