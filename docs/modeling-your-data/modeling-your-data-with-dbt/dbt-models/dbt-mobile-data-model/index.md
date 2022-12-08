---
title: "The Mobile data model"
date: "2022-05-14"
sidebar_position: 102
---
# Snowplow Mobile Package

**The package source code can be found in the [snowplow/dbt-snowplow-mobile repo](https://github.com/snowplow/dbt-snowplow-mobile), and the docs for the [model design here](https://snowplow.github.io/dbt-snowplow-mobile/#!/overview/snowplow_mobile).** 

The package contains a fully incremental model that transforms raw mobile event data generated by the [Snowplow iOS](docs/collecting-data/collecting-from-own-applications/mobile-trackers/previous-versions/objective-c-tracker/index.md) or [android trackers](/docs/collecting-data/collecting-from-own-applications/mobile-trackers/previous-versions/android-tracker/index.md) into a series of derived tables of varying levels of aggregation.

The Snowplow mobile data model aggregates Snowplow's out-of-the-box mobile events to create a set of derived tables - screen views, sessions, and users. These contain many useful dimensions, as well as calculated measures such as screen views per session.

![](images/Screenshot-2022-03-25-at-10.50.33.png)



## Overview

This model consists of a series of modules, each producing a table which serves as the input to the next module. The 'standard' modules are:

- Base: Performs the incremental logic, outputting the table `snowplow_mobile_base_events_this_run` which contains a de-duped data set of all events required for the current run of the model.
- Screen Views: Aggregates event level data to a screen view level, on `screen_view_id`, outputting the table `snowplow_mobile_screen_views`.
- Sessions: Aggregates screen view level data to a session level, on `session_id`, outputting the table `snowplow_mobile_sessions`.
- Users: Aggregates session level data to a users level, on `device_user_id`, outputting the table `snowplow_mobile_users`.
- User Mapping: Provides a mapping between user identifiers, `device_user_id` and `user_id`, outputting the table `snowplow_mobile_user_mapping`. This can be used for session stitching.

## Contexts

The following contexts can be enabled depending on your tracker configuration:

- Mobile context
- Geolocation context
- Application context
- Screen context

By default they are disabled. They can be enabled by configuring the `dbt_project.yml` file and setting the appropriate `snowplow__enable_{context_type}_context` variable to `true`.

## Optional Modules

Currently the App Errors module, used for crash reporting, is the only optional module. More will be added in the future as the tracker's functionality expands.

### App Errors

Assuming your tracker is capturing `application_error` events, the module can be enabled by configuring the `dbt_project.yml` file:

```yaml
vars:
  snowplow_mobile:
    snowplow__enable_app_errors_module: true
```