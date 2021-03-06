# Overview

Module Name: Viewability

Purpose: Track when a given HTML element becomes viewable

Maintainer: atrajkovic@magnite.com

# Configuration

Module does not need any configuration, as long as you include it in your PBJS bundle.
Viewability module has only two functions `startMeasurement` and `stopMeasurement` which can be used to enable more complex viewability measurements. Since it allows tracking from within creative (possibly inside a safe frame) this module registers a message listener, for messages with a format that is described bellow.

## `startMeasurement`

| startMeasurement Arg Object | Scope | Type | Description | Example |
| --------------------- | -------- | ------------ | -------------------------------------------------------------------------------- | --------- |
| vid | Required | String | Unique viewability identifier, used to reference particular observer | `"ae0f9"` |
| element | Required | HTMLElement |  Reference to an HTML element that needs to be tracked | `document.getElementById('test_div')` |
| tracker | Required | ViewabilityTracker | How viewaility event is communicated back to the parties of interest | `{ method: 'img', value: 'http://my.tracker/123' }` |
| criteria | Required | ViewabilityCriteria| Defines custom viewability criteria using the threshold and duration provided  | `{ inViewThreshold: 0.5, timeInView: 1000 }` |

| ViewabilityTracker | Scope | Type | Description | Example |
| --------------------- | -------- | ------------ | -------------------------------------------------------------------------------- | --------- |
| method | Required | String | Type of method for Tracker | `'img' OR 'js' OR 'callback'` |
| value | Required | String | URL string for 'img' and 'js' Trackers, or a function for 'callback' Tracker | `'http://my.tracker/123'` |

| ViewabilityCriteria | Scope | Type | Description | Example |
| --------------------- | -------- | ------------ | -------------------------------------------------------------------------------- | --------- |
| inViewThreshold | Required | Number | Represents a percentage threshold for the Element to be registered as in view | `0.5` |
| timeInView | Required | Number | Number of milliseconds that a given element needs to be in view continuously, above the threshold | `1000` |

## Please Note:
- `vid` allows for multiple trackers, with different criteria to be registered for a given HTML element, independently. It's not autogenerated by `startMeasurement()`, it needs to be provided by the caller so that it doesn't have to be posted back to the source iframe (in case viewability is started from within the creative).
- In case of 'callback' method, HTML element is being passed back to the callback function.
- When a tracker needs to be started, without direct access to pbjs, postMessage mechanism can be used to invoke `startMeasurement`, with a following payload: `vid`, `tracker` and `criteria` as described above, but also with `message: 'Prebid Viewability'` and `action: 'startMeasurement'`. Optionally payload can provide `elementId`, if available at that time (for ad servers where name of the iframe is known, or adservers that render outside an iframe). If `elementId` is not provided, viewability module will try to find the iframe that corresponds to the message source. 


## `stopMeasurement`

| stopMeasurement Arg Object | Scope | Type | Description | Example |
| --------------------- | -------- | ------------ | -------------------------------------------------------------------------------- | --------- |
| vid | Required | String | Unique viewability identifier, referencing an already started viewability tracker. | `"ae0f9"` |

## Please Note:
- When a tracker needs to be stopped, without direct access to pbjs, postMessage mechanism can be used here as well. To invoke `stopMeasurement`, you provide the payload with `vid`, `message: 'Prebid Viewability'` and `action: 'stopMeasurement`. Check the example bellow.

# Examples

## Example of starting a viewability measurement, when you have direct access to pbjs
```
pbjs.viewability.startMeasurement(
    'ae0f9', 
    document.getElementById('test_div'),
    { method: 'img', value: 'http://my.tracker/123' },
    { inViewThreshold: 0.5, timeInView: 1000 }
);
```

## Example of starting a viewability measurement from within a rendered creative
```
let viewabilityRecord = {
    vid: 'ae0f9',
    tracker: { method: 'img', value: 'http://my.tracker/123'},
    criteria: { inViewThreshold: 0.5, timeInView: 1000 },
    message: 'Prebid Viewability',
    action: 'startMeasurement'
}

window.parent.postMessage(JSON.stringify(viewabilityRecord), '*');
```

## Example of stopping the viewability measurement, when you have direct access to pbjs
```
pbjs.viewability.stopMeasurement('ae0f9');
```

## Example of stopping the viewability measurement from within a rendered creative
```
let viewabilityRecord = {
    vid: 'ae0f9',
    message: 'Prebid Viewability',
    action: 'stopMeasurement'
}

window.parent.postMessage(JSON.stringify(viewabilityRecord), '*');
```
