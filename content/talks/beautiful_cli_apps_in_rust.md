---
title: 'Beautiful CLI Apps in Rust'
date: "2022-10-18T14:45:00"
categories: ["workshop"]
tags: ["opensuse", "packaging"]
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

https://danyspin97.org/talks/beautiful_cli_apps_in_rust

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

Read the OpenWeather API key from a config file ![:i](fas fa-arrow-right) **serde** and **toml**

Fetch and parse the data ![:i](fas fa-arrow-right)  **reqwest** and **serde-json**

Print it to the user in a nice way ![:i](fas fa-arrow-right) **termcolor**

---
background-image: url(/img/beautiful_cli_apps_in_rust/codemotion_page.png)
class: left, middle

## Clap

Powerful and flexible command line argument library

```rust
use clap::Parser;

#[derive(Parser)]
#[command(version, about)]
struct Args {
   city: String,
}

fn main() {
    let args = Args::try_parse().unwrap();
```

---
background-image: url(/img/beautiful_cli_apps_in_rust/codemotion_page.png)
class: left, middle

## xdg

Library that implements the [XDG BaseDirectories specification](http://standards.freedesktop.org/basedir-spec/basedir-spec-latest.html)

```rust
    let xdg_dirs = xdg::BaseDirectories::with_prefix("weather-cli").unwrap();
    let config_file = xdg_dirs.find_config_file("weather-cli.conf")
        .expect("could not find configuration file");
```

---
background-image: url(/img/beautiful_cli_apps_in_rust/codemotion_page.png)
class: left, middle

## serde

De-facto standard serialization and deserialization library

```rust
use serde::Deserialize;

#[derive(Deserialize)]
struct Config {
  api_key: String
}
```

---
background-image: url(/img/beautiful_cli_apps_in_rust/codemotion_page.png)
class: left, middle

## toml

Implement TOML data format for serde

```rust
    use std::fs;

    let config = toml::from_str(fs::read_to_string(config_file)
        .expect("could not read config file"))
        .expect("could not deserialize config file");
```

---
background-image: url(/img/beautiful_cli_apps_in_rust/codemotion_page.png)
class: left, middle

## reqwest (1/2)

Provides a convenient, higher-level HTTP Client

```rust
    const GEO_URL = "http://api.openweathermap.org/geo/1.0/direct?limit=1";

    use reqwest::blocking::Client;
    let client = Client::new();

    let params = [("appid", "bar"), ("q", &args.city)];
    let res = client.post(GEO_URL)
        .body(params)
        .send()
        .expect(format!("could not get geodata about {}", args.city));
```

[API documentation](https://openweathermap.org/api/geocoding-api)

---
background-image: url(/img/beautiful_cli_apps_in_rust/codemotion_page.png)
class: left, middle

## serde-json (1/2)

Implements JSON data format for serde

```rust
    use serde_json::Value;

    let json_value : Value = serde_json::from_str(res)
        .expect("could not parse JSON data");
    let geo_data = json_value[0];
    let lat = geo_data["lat"];
    let lon = geo_data["lon"];
```

---
background-image: url(/img/beautiful_cli_apps_in_rust/codemotion_page.png)
class: left, middle

## reqwest (2/2)

```rust
    const WEATHER_URL = "https://api.openweathermap.org/data/2.5/weather";

    let params = [("appid", "bar"), ("lat", &lat), ("lon", &lon)];
    let res = client.post(WEATHER_URL)
        .body(params)
        .send()
        .expect(format!("could not get weather for {}", args.city));
```

[API documentation](https://openweathermap.org/current)

---
background-image: url(/img/beautiful_cli_apps_in_rust/codemotion_page.png)
class: left, middle


## serde-json (2/2)

```rust
    let json_value : Value = serde_json::from_str(res)
        .expect("could not parse JSON data");
    let weather = json_value["weather"][0];
```

---
background-image: url(/img/beautiful_cli_apps_in_rust/codemotion_page.png)
class: left, middle

## Printing the weather

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

