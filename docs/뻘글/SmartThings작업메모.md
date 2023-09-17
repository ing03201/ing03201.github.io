---
layout: default
title: Smarthings 작업 메모
parent: 뻘글
permalink: /뻘글/SmartThingsTask
---

#  SmartThings 작업 메모

## Capabilities


## st.mdns

```lua
-- scan for Hue bridges on the local network
local discover_responses = mdns.discover("_hue._tcp", "local") or {}

for idx, found in ipairs(discover_responses.found) do
  -- sanity check that the answer contains a response to the correct service type,
  -- and we only want to process ipv4
  if found ~= nil and found.service_info.name == "_hue._tcp"
      and not net_utils.validate_ipv4_string(found.host_info.address) then
    -- process response
  end
end
```
