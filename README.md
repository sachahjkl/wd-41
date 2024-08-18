# (w)eb (d)evelopment-(40)
[![Go Report Card](https://goreportcard.com/badge/github.com/baalimago/wd-41)](https://goreportcard.com/report/github.com/baalimago/wd-41)
[![wakatime](https://wakatime.com/badge/user/018cc8d2-3fd9-47ef-81dc-e4ad645d5f34/project/3bc921ec-dc23-4222-bf00-578f2eda0cbd.svg)](https://wakatime.com/badge/user/018cc8d2-3fd9-47ef-81dc-e4ad645d5f34/project/3bc921ec-dc23-4222-bf00-578f2eda0cbd)

Test coverage: 73.640% 😌👏

This is a very simple static webserver which hot-reloads your web-browser on changes to the statically hosted webserver.

## Usage
`wd-41 s|serve <relative directory>` or `wd-41 s|serve`

## Getting started
```bash
go install github.com/baalimago/wd-41@latest
```

You may also use the setup script:
```bash
curl -fsSL https://raw.githubusercontent.com/baalimago/wd-41/main/setup.sh | sh
```

## Architecture
1. First the content of the website is copied to a temporary directory, this is the _mirrored content_ 
1. Every mirrored file is inspectd for type, if it's text/html, a `delta-streamer.js` script is injected
1. The web server is started, hosting the _mirrored_ content
1. The `delta-streamer.js` in turn sets up a websocket connection to the wd-41 webserver
1. The original file system is monitored, on any file changes:
    1. the new file is copied to the mirror (including injections)
    1. the file name is propagated to the browser via the websocket
1. The `delta-streamer.js` script then checks if the current window origin is the updated file. If so, it reloads the page.
```
       ┌───────────────┐                                                 
       │ Web Developer │                                                 
       └───────┬───────┘                                                 
               │                                                         
       [writes <content>]                                                
               │                                                         
               ▼                                                         
 ┌─────────────────────────────┐        ┌─────────────────────┐          
 │ website-directory/<content> │        │ file system notify  │          
 └─────────────┬───────────────┘        └─────────┬───────────┘          
               │                                  │                      
               │                      [update mirrored content]          
               ▼                                  │                      
     ┌────────────────────┐                       │                      
     │ ws-script injector │◄──────────────────────┘                      
     └─────────┬──────────┘                                              
               │                                                         
               │                                                         
               ▼                                                         
   ┌────────────────────────┐                                            
   │ tmp-abcd1234/<content> │                                            
   └───────────┬────────────┘                                            
               │                                                         
       [serves <content>]                                                
               │                               ┌────────────────────────┐
               ▼                               │         Browser        │
┌──────────────────────────────┐               │                        │
│          Web Server          │               │  ┌────┐  ┌───────────┐ │
│ [localhost:<port>/<content>] │◄───[reload────┼─►│ ws │  │ <content> │ │
└──────────────────────────────┘     page]     │  └────┘  └───────────┘ │
                                               │                        │
                                               └────────────────────────┘
```
