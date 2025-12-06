# Deep Logic & Data Summary

This document details the data flow, tool chains, and exact signal structures for the 3 key platforms.

## 1. Amazon (Shopping)
**Objective:** Track potential purchases and lingering interest.

| Process Step | Tool Chain | Data Signal (TEST_SIGNAL Payload) | Logic & Outcome |
| :--- | :--- | :--- | :--- |
| **1. Detection** | `userBehaviour.js` (Listener) <br> `SiteDictionary.js` (Context) <br> `DomMiner.js` (Emitter) | **Operation:** `SCROLL` <br> **Metadata:** <br> `{ "label": "PRODUCT_PAGE", "activity": "SHOPPING", "siteName": "AMAZON" }` | **Raw Event:** Detected user scrolling on a product page. |
| **2. Logic** | `ActionClassifier.js` | *Internal Event: HybridOperationMatch* | **Rule:** If `SCROLL` on `PRODUCT_PAGE` > 2s <br> **Action:** Emits `ANALYSIS` |
| **3. Intervention** | `ActivityInferencer.js` <br> `Perplexity API` | **Input to AI:** <br> `"- ANALYSIS (Label: PRODUCT_PAGE)"` | **AI Response:** "The user is analyzing features, signaling high purchase intent." |

## 2. Google (Search)
**Objective:** Profile user interests based on query formation.

| Process Step | Tool Chain | Data Signal (TEST_SIGNAL Payload) | Logic & Outcome |
| :--- | :--- | :--- | :--- |
| **1. Detection** | `userBehaviour.js` (Listener) <br> `SiteDictionary.js` (Context) <br> `DomMiner.js` (Emitter) | **Operation:** `SEARCH_INPUT` <br> **Metadata:** <br> `{ "label": "SEARCH_ENGINE", "activity": "QUERY_FORMULATION", "siteName": "GOOGLE" }` | **Raw Event:** Detected typing in a search bar (not password). |
| **2. Logic** | `ActionClassifier.js` | *Internal Event: HybridOperationMatch* | **Rule:** If `SEARCH_INPUT` detected <br> **Action:** Emits `QUERY_FORMULATION` |
| **3. Intervention** | `ActivityInferencer.js` <br> `Perplexity API` | **Input to AI:** <br> `"- QUERY_FORMULATION (Label: SEARCH_ENGINE)"` | **AI Response:** "Typing detailed queries indicates an 'Information Foraging' phase." |

## 3. YouTube (Entertainment)
**Objective:** Measure engagement and passive consumption.

| Process Step | Tool Chain | Data Signal (TEST_SIGNAL Payload) | Logic & Outcome |
| :--- | :--- | :--- | :--- |
| **1. Detection** | `userBehaviour.js` (Listener) <br> `SiteDictionary.js` (Context) <br> `DomMiner.js` (Emitter) | **Operation:** `CLICK` <br> **Metadata:** <br> `{ "label": "VIDEO_PLAYER", "activity": "ENTERTAINMENT", "siteName": "YOUTUBE" }` | **Raw Event:** Interaction with the video player area. |
| **2. Logic** | `ActionClassifier.js` | *Internal Event: HybridOperationMatch* | **Rule:** If `CLICK` on `VIDEO_PLAYER` <br> **Action:** Emits `SELECTION` |
| **3. Intervention** | `ActivityInferencer.js` <br> `Perplexity API` | **Input to AI:** <br> `"- SELECTION (Label: VIDEO_PLAYER)"` | **AI Response:** "Clicking play/pause signals active engagement vs passive watching." |

## Tool Definitions
*   **`userBehaviour.js`**: Third-party library that attaches `mousedown`, `scroll`, `keyup` listeners to the window.
*   **`SiteDictionary.js`**: Custom library that checks `window.location.href` and DOM Selectors (e.g. `#movie_player`) to assign the `label`.
*   **`DomMiner.js`**: Bridges the two above. It runs the loop, calls `SiteDictionary`, and `console.log("TEST_SIGNAL", ...)`
*   **`ActionClassifier.js`**: The local "Brain". It applies rules (e.g. "Is this scroll long enough?") to filter noise.
*   **`Perplexity API`**: The Cloud "Analyst". It takes the clean list of Actions and writes the human-readable explanation.
