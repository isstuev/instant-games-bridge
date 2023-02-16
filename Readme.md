# Instant Games Bridge
One SDK for cross-platform publishing HTML5 games.

Supported platforms:
+ [VK.COM](https://vk.com)
+ [Yandex Games](https://yandex.com/games)
+ [Crazy Games](https://crazygames.com)

Plugins for game engines:
+ [Construct 3](https://github.com/mewtongames/instant-games-bridge-construct)
+ [Unity](https://github.com/mewtongames/instant-games-bridge-unity)
+ [Defold](https://github.com/mewtongames/instant-games-bridge-defold)
+ [Godot](https://github.com/mewtongames/instant-games-bridge-godot)

Roadmap: https://trello.com/b/NjF29vTW.

Join community: https://t.me/instant_games_bridge.

## Usage
+ [Setup](#setup)
+ [Platform](#platform)
+ [Device](#device)
+ [Player](#player)
+ [Game](#game)
+ [Storage](#storage)
+ [Advertisement](#advertisement)
+ [Social](#social)
+ [Leaderboard](#leaderboard)

### Setup
First you need to initialize the SDK:
```html
<script src="./instant-games-bridge.js"></script>
<script>
    bridge.initialize()
        .then(() => {
            // Initialized. You can use other methods.
        })
        .catch(error => {
            // Error
        })
    
    // You can forcibly set needed platform ID and skip the internal detection
    bridge.initialize({ forciblySetPlatformId: bridge.PLATFORM_ID.MOCK })
</script>
```

### Platform
```js
// ID of current platform ('vk', 'yandex', 'crazy_games', 'mock')
bridge.platform.id

// Platform native SDK
bridge.platform.sdk

// If platform provides information - this is the user language on platform. 
// If not - this is the language of the user's browser.
bridge.platform.language

// The value of the payload parameter from the url. Examples:
// VK: vk.com/app8056947#your-info
// Yandex: yandex.com/games/play/183100?payload=your-info
// CrazyGames: crazygames.com/game/example?payload=your-info
// Mock: site.com/game?payload=your-info
bridge.platform.payload

// Message to the platform. Supported only on CrazyGames.
// Possible values: 'game_loading_started', 'game_loading_stopped', 'gameplay_started', 'gameplay_stopped', 'player_got_achievement'
bridge.platform.sendMessage('player_got_achievement')
```

### Device
```js
// Possible values: 'mobile', 'tablet', 'desktop', 'tv'
bridge.device.type
```

### Player
```js
// VK, Yandex: true
// CrazyGames: false
bridge.player.isAuthorizationSupported

// VK: true, Yandex: true/false
// CrazyGames: false
bridge.player.isAuthorized

// If player is authorized
bridge.player.id

// If player is authorized (Yandex: and allowed access to this information)
bridge.player.name
bridge.player.photos // Array of player photos, sorted in order of increasing photo size

// If authorization is supported and player is not authorized
let authorizationOptions = {
    'yandex': {
        scopes: true // Request access to name and photo
    }
}
bridge.player.authorize(authorizationOptions)
    .then(() => {
        // Success
    })
    .catch(error => {
        // Error
    })
```

### Game
```js
// Current visibility state ('visible', 'hidden')
bridge.game.visibilityState

// Fired when visibility state changed ('visible', 'hidden')
// For example: you can play/pause music here 
bridge.game.on(bridge.EVENT_NAME.VISIBILITY_STATE_CHANGED, state => console.log('Visibility state:', state))
```

### Storage
```js
// Current platform storage type ('local_storage', 'platform_internal')
bridge.storage.defaultType

// Check if the storage supported
bridge.storage.isSupported(bridge.STORAGE_TYPE.LOCAL_STORAGE)
bridge.storage.isSupported(bridge.STORAGE_TYPE.PLATFORM_INTERNAL)

// Get data from storage
bridge.storage.get('key')
    .then(data => {
        // Data has been received and you can work with them
        // data = null if there is no data for this key
        console.log('Data:', data)
    })
    .catch(error => {
        // Error
    })

// Set game data in storage
bridge.storage.set('key', 'value')
    .then(() => {
        // Success
    })
    .catch(error => {
        // Error
    })

// Delete game data from storage
bridge.storage.delete('key')
    .then(() => {
        // Success
    })
    .catch(error => {
        // Error
    })

/* -- -- -- Different Storage Types -- -- -- */
// You can choose storage type for each platform separately:
let options = {
    'vk': bridge.STORAGE_TYPE.PLATFORM_INTERNAL,
    'yandex': bridge.STORAGE_TYPE.LOCAL_STORAGE,
    'crazy_games': bridge.STORAGE_TYPE.LOCAL_STORAGE
}
bridge.storage.get('key', options)
bridge.storage.set('key', 'value', options)
bridge.storage.delete('key', options)

// Or common to all platforms:
let storageType = bridge.STORAGE_TYPE.LOCAL_STORAGE
bridge.storage.get('key', storageType)
bridge.storage.set('key', 'value', storageType)
bridge.storage.delete('key', storageType)

/* -- -- -- Multiple keys and values -- -- -- */
// You can send an array of keys and values
bridge.storage.get(['key_1', 'key2'])
bridge.storage.set(['key_1', 'key2'], ['value_1', 'value_2'])
bridge.storage.delete(['key_1', 'key2'])
```

### Advertisement
If you want to show banners on VK — add [bridge-vk-banner-extension](https://github.com/instant-games-bridge/instant-games-bridge-vk-banner-extension) (+482kb).
```js
/* -- -- -- Banners -- -- -- */
bridge.advertisement.isBannerSupported

// Fired when banner state changed ('shown', 'hidden', 'failed')
bridge.advertisement.on(bridge.EVENT_NAME.BANNER_STATE_CHANGED, state => console.log('Banner state:', state))

let bannerOptions = {
    'vk': {
        position: 'top' // Default = bottom
    },
    'crazy_games': {
        containerId: 'div-container-id'
    }
}
bridge.advertisement.showBanner(bannerOptions)
bridge.advertisement.hideBanner()

/* -- -- -- Delays Between Interstitials -- -- -- */
bridge.advertisement.minimumDelayBetweenInterstitial // Default = 60 seconds

// You can override minimum delay. You can use platform specific delays:
let delayOptions = {
    'vk': 30,
    'yandex': 60,
    'crazy_games': 60,
    'mock': 0
}
// Or common to all platforms:
let delayOptions = 60
bridge.advertisement.setMinimumDelayBetweenInterstitial(delayOptions)

/* -- -- -- Interstitial -- -- -- */
// Fired when interstitial state changed ('opened', 'closed', 'failed')
bridge.advertisement.on(bridge.EVENT_NAME.INTERSTITIAL_STATE_CHANGED, state => console.log('Interstitial state:', state))

// Optional parameter. You can use platform specific ignoring:
let interstitialOptions = {
    'vk': {
        ignoreDelay: true
    },
    'yandex': {
        ignoreDelay: false
    },
    'crazy_games': {
        ignoreDelay: false
    }
}
// Or common to all platforms:
let interstitialOptions = {
    ignoreDelay: true // Default = false
}
// Request to show interstitial ads
bridge.advertisement.showInterstitial(interstitialOptions)

/* -- -- -- Rewarded Video -- -- -- */
// Fired when rewarded video state changed ('opened', 'rewarded', 'closed', 'failed')
// It is recommended to give a reward when the state is 'rewarded'
bridge.advertisement.on(bridge.EVENT_NAME.REWARDED_STATE_CHANGED, state => console.log('Rewarded state:', state))

// Request to show rewarded video ads
bridge.advertisement.showRewarded()
```

### Social
```js
// VK: true
// Yandex, CrazyGames: false
bridge.social.isShareSupported
bridge.social.isJoinCommunitySupported
bridge.social.isInviteFriendsSupported
bridge.social.isCreatePostSupported
bridge.social.isAddToFavoritesSupported

// VK, Yandex: partial supported
// CrazyGames: false
bridge.social.isAddToHomeScreenSupported

// VK, CrazyGames: false
// Yandex: true
bridge.social.isRateSupported

let shareOptions = {
    'vk': {
        link: 'https://vk.com/wordle.game'
    }
}
bridge.social.share(shareOptions)
    .then(() => {
        // Success
    })
    .catch(error => {
        // Error
    })

let joinCommunityOptions = {
    'vk': {
        groupId: '199747461'
    }
}
bridge.social.joinCommunity(joinCommunityOptions)
    .then(() => {
        // Success
    })
    .catch(error => {
        // Error
    })

bridge.social.inviteFriends()
    .then(() => {
        // Success
    })
    .catch(error => {
        // Error
    })

let createPostOptions = {
    'vk': {
        message: 'Hello world!',
        attachments: 'photo-199747461_457239629'
    }
}
bridge.social.createPost(createPostOptions)
    .then(() => {
        // Success
    })
    .catch(error => {
        // Error
    })

bridge.social.addToHomeScreen()
    .then(() => {
        // Success
    })
    .catch(error => {
        // Error
    })

bridge.social.addToFavorites()
    .then(() => {
        // Success
    })
    .catch(error => {
        // Error
    })

bridge.social.rate()
    .then(() => {
        // Success
    })
    .catch(error => {
        // Error
    })
```

### Leaderboard
```js
// VK, Yandex: true
// CrazyGames: false
bridge.leaderboard.isSupported

// VK: true
// Yandex, CrazyGames: false
bridge.leaderboard.isNativePopupSupported

// VK, CrazyGames: false
// Yandex: true
bridge.leaderboard.isMultipleBoardsSupported
bridge.leaderboard.isSetScoreSupported
bridge.leaderboard.isGetScoreSupported
bridge.leaderboard.isGetEntriesSupported

let setScoreOptions = {
    'yandex': {
        leaderboardName: 'YOU_LEADERBOARD_NAME',
        score: 42
    }
}
bridge.leaderboard.setScore(setScoreOptions)
    .then(() => {
        // Success
    })
    .catch(error => {
        // Error
    })

let getScoreOptions = {
    'yandex': {
        leaderboardName: 'YOU_LEADERBOARD_NAME',
    }
}
bridge.leaderboard.getScore(getScoreOptions)
    .then(score => {
        // Success
        console.log(score)
    })
    .catch(error => {
        // Error
    })

let getEntriesOptions = {
    'yandex': {
        leaderboardName: 'YOU_LEADERBOARD_NAME',
        includeUser: true, // Default = false
        quantityAround: 10, // Default = 5
        quantityTop: 10 // Default = 5
    }
}
bridge.leaderboard.getEntries(getEntriesOptions)
    .then(entries => {
        // Success
        entries.forEach(e => {
            console.log('ID: ' + e.id + ', name: ' + e.name + ', score: ' + e.score + ', rank: ' + e.rank + ', small photo: ' + e.photos[0])
        })
    })
    .catch(error => {
        // Error
    })

let showNativePopupOptions = {
    'vk': {
        userResult: 42,
        global: true // Default = false
    }
}
bridge.leaderboard.showNativePopup(showNativePopupOptions)
    .then(() => {
        // Success
    })
    .catch(error => {
        // Error
    })
```