# [ARO](http://interruptedlobster.github.io/)
Augmented Reality Objects

[![Current Sprint](https://badge.waffle.io/InterruptedLobster/findAR.png?label=ready&title=Current%20Sprint)](https://waffle.io/InterruptedLobster/findAR)
[![Ready For Review](https://badge.waffle.io/InterruptedLobster/findAR.png?label=review%20me&title=Review%20Ready)](https://waffle.io/InterruptedLobster/findAR)
[![Dependency Status](https://david-dm.org/kkragenbrink/jasmine.svg)](https://david-dm.org/InterruptedLobster/findAR)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](https://github.com/kkragenbrink/jasmine/blob/master/LICENSE.txt)


## Installation:
```
npm install
npm install -g rnpm
rnpm link
```
### Simulator:
```
react-native run-ios
```
### Build:
```
open ios/findAR.xcodeproj
⌘R
```

## Directory Structure
```
findAR/                                # Root Directory
  app/                                 # React file structure
    actions/                           # Redux actions
    assets/                            # Graphics
    components/                        # Custom react components
      AR.js                            # Displays camera and 3D overlay
      FriendList.js                    # Shows app-using friends, accepts onPress prop
      FriendListItem.js                # Renders individual friend in list
      Map.js                           # Renders map view
      PinCallout.js                    # Renders pin popout
      PinEditButton.js                 # Renders edit button on clicking a pin on map
      PinList.js                       # Renders list of saved locations
      PinListItem.js                   # Renders individual location, handles click events
      ViewContainer.js                 # Renders AR, Map, or PinList depending on mode
    constants/                         # Redux action types
    containers/                        # Redux smart containers
      ...dropNewPin.js                 # Handles logic to drop new pin
      ...FBLogin.js                    # Handles facebook login
      ...viewContainer.js              # Maps redux store to props for ViewContainer.js
    lib/                               # Utility functions
      db/                              # Firebase helpers
      orientation/                     # Device orientation / compass direction helpers
      threejs/                         # THREE.js models and helpers
    reducers/                          # Redux reducers
  ios/                                 # Xcode/native components
  index.ios.js                         # Entry point for React Native app, handles routing
```

## React Component Hierarchy
```
  <Provider>                           # index.ios.js -- Redux
    <Router>                           # index.ios.js -- react-native-router-flux
      <Scene key="root">             
        <Scene initial key="login">
          <LogIn />
        </Scene>
        <Scene key="view">
          <ViewContainer>              # container_viewContainer.js
            <AR>                       # Conditionally rendered -- AR.js
              <Camera />               # react-native-camera
              <WebViewBridge />        # react-native-webview-bridge -- overlays threejs on transparent DOM
            </AR>
            <Map>                      # Conditionally rendered -- Map.js
              <MapView>                # react-native-maps
                <MapView.Marker>       # Many of these! Each represents a location or friend, generated by
                                       # renderMarkers() or renderFriends()
                  <PinCallout />       # PinCallout.js
                </MapView.Marker>
              </MapView>
              <PinEditButton />        # Conditionally rendered -- PinEditButton.js
              <Button />               # Centers on user's current location
            </Map>
            <PinList>                  # Conditionally rendered -- PinList.js
              <PinListItem>            # Lots of these! Handles onPress for each item, renders distance and
                                       # name of friend that shared. Uses react-redux-router-flux to switch
                                       # to 'friends' scene when the 'share' option is pressed.
              </PinListItem>
            </PinList>
            <ViewModeButton />         # Conditionally rendered
            <DropNewPinButton />       # container_dropNewPin.js
          </ViewContainer>
        </Scene>
        <Scene key="friends">
          <FriendList>                 # FriendList.js -- takes onPress as prop
            <FriendListItem />         # FriendListItem.js  -- handles onPress, routes back to previous scene
          </FriendList>
        </Scene>
      </Scene>
    </Router>
  </Provider>
```
## React Component Diagram ( View Scene )
![Component Diagram](http://interruptedlobster.github.io/assets/view_scene.png)

## Firebase Schema

Variables starting with $ are not the actual variables used in the application, but refer
to keys that may differ from object to object.
```
InterruptedLobster: {
  $id: {                               # The facebook user ID of a given account.
    email: STRING,
    id: STRING                         # Matches $id
    name: STRING,
    picture: STRING,
    pins: {                            # The locations this person has saved.
      $pid: {                          # The ID of a given location
        alertedYet: BOOL,              # Not set unless pin is shared. False until user is notified.
        friend: {                      # Not set unless pin is shared
          email: STRING,
          name: STRING,
          id: STRING,                  # Facebook user ID of friend who shared the pin
          picture: STRING,             # URL path to profile picture
        },
        id: STRING,                    # Matches $pid
        latitude: NUMBER,
        longitude: NUMBER,
        title: STRING,
      }
    },
    recent: {
      $n: STRING,                      # Key is array index, value is $pid referring to recently placed pin
    },
  },
  currLoc: {                           # Denormalized to allow quick access to broadcasted friends' locations
    $id: {                             # Facebook user ID of users broadcasting their current location
      longitude: NUMBER,
      latitude: NUMBER,
    }
  },

}
```

## Redux Store

Variables starting with $ are not the actual variables used in the application, but refer
to keys that may differ from object to object.
```
Store: {
  pins: {                              # A hash table of pins
    $pid: {                            # String representing a pin's ID.
      alertedYet: BOOL,                # Not set unless pin is shared. False until user is notified.
      friend: {                        # Not set unless pin is shared
        email: STRING,
        name: STRING,
        id: STRING,                    # Facebook user ID of friend who shared the pin
        picture: STRING,               # URL path to profile picture
      },
      id: STRING,                      # Matches $pid
      latitude: NUMBER,
      longitude: NUMBER,
      title: STRING,
    }
  },
  recent: [                            # Array of IDs of recently placed pins
    $pid                               # A pin's ID
  ],
  user: {
    name: STRING,
    email: STRING,
    id: STRING,                        # Facebook ID
    picture: STRING,                   # URL path to profile picture
  },
  friends: {
    $uid: {                            # String representing a friend's Facebook ID.
      name: STRING,
      id: STRING,                      # Matches $uid 
      picture: STRING,                 # URL path to friend's profile picture
    }
  },
  targetPin: {                         # Defaults as { id: 0 } to resolve some edge cases.
    id: STRING,                        # Matches a $pid
    latitude: NUMBER,
    longitude: NUMBER,
    title: STRING,
  }
}
```

## Architecture Diagram:
![Architecture Diagram](http://interruptedlobster.github.io/assets/high-level.png)
