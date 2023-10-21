---
layout: default
title: Smarthings 작업 메모
parent: 뻘글
permalink: /뻘글/SmartThingsTask
---

#  SmartThings 작업 메모

## Custom Capabilities
스마트싱스 device의 속성을 **Capability**라고 한다.
나만의 capabilities를 만들고자한다면 Smartthings cli로 만들수있다.


## 빌드 방법
```Shell
## edge channel을 만들었다 가정했을때
smartthings edge:drivers:package [path_to_dir]
smartthings edge:channels:assign [driver_id] [version]
smartthings edge:drivers:install [driver_id]

## 디버깅
smartthings edge:drivers:logcat
```
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

## Should_continue
- 기기찾기를 하지 않아도 알아서 자동으로 검색해서 연결을 해주는 함수 ( 백그라운드가 계속 돌고있음 )