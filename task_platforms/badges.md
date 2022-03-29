---
layout: page
title: Badges
nav_order: 80
parent: Interactions with Other Platforms
---

# Badges

Badges are used to prove to a destination platform that you have reached some objectives in another plaform. This typically allows you to get a specific extra-access to a protected ressource on the destination platform. It is used across many apps, using the "login-module" or not.

## Sample workflow

{% include badge_workflow.html %}

## Badge processing by the backend

In practice, for the backend, badges are just interpreted as group memberships, which may give some specific permissions using the usual permission mechanism on the backend. The badges are transmitted from the login-module to the backend via the update of the profile. The `aBadges` attribute of the profile is an array of entries with the following format:

```js
{
  url: string, // unique identifier of the badge
  badge_info: {
    name: string, // display name of the badge, also used as group name
    group_path: [
      // hierarchy of parent groups with their name, a unique url for each, and whether the user should be manager of those groups as well
      // goes from highest ancestor to direct parent
      { url: string, name: string, manager: bool },
      ...
    ]
  },
  manager: bool, // whether the user should be manager of the group of owners of that badge
  code: string, // ignored by backend, probably won't be sent anymore
  data: any // ignored by backend, can be anything
}
```

Once the profile is processed, the backend must check if the user belongs to all groups specified in the badges. Groups are identified by their `url` with are stored in `groups.text_id`. So, for each group of the list:
- if the group exists, and the user is already member (or manager is `manager` is `true`) of it : do nothing
- if the group exists, and the user is not already member (or manager is `manager` is `true`) of it : make him member of the group
- if the group does not exist, create a group with `badge_info.name` as name and type "Other", add it to the group identified by `url` of the last element from `badge_info.group_path`. If this later group does not exist, create it (with the given name, and current user managership) and put it the previous group from `badge_info.group_path`, etc.
