# Skill: Food & Dining Recommendation Sub-Agent

## Overview
This skill outlines the functional specification for a simplified **Food & Dining Sub-Agent**. The agent's sole purpose is to receive a hotel location from a Hotel Sub-Agent/Master Agent, scan the vicinity, and return up to **3 top dining recommendations** that satisfy two strict criteria.

---

## 1. Core Constraints (Hard Criteria)
To be eligible for recommendation, a location **must** meet both parameters:
1. **Google Rating:** Strict minimum score above 4.5 (`rating > 4.5`).
2. **Proximity:** Walking distance less than 500 meters from the provided hotel coordinates (`distance < 500m`).

---

## 2. Agent Workflow

1. **Receive Context:** Read incoming JSON event containing the selected hotel's location/coordinates.
2. **Query & Filter:** Fetch nearby food places and eliminate any options with a Google rating $\le 4.5$ or distance $\ge 500\text{m}$.
3. **Select Top 3:** Rank valid options using a combined metric of highest Google rating and shortest distance, selecting up to 3 best places.
4. **Return Response:** Output structured JSON back to the calling agent.

---

## 3. Data Contracts

### Input Payload (From Hotel/Master Agent)
```json
{
  "event_type": "REQUEST_DINING_RECOMMENDATIONS",
  "hotel_location": {
    "name": "Grand Hotel",
    "coordinates": {
      "lat": 32.0853,
      "lng": 34.7818
    }
  }
}
```

### Output Payload (Top 3 Recommendations)
```json
{
  "event_type": "DINING_RECOMMENDATIONS_PROVIDED",
  "hotel_name": "Grand Hotel",
  "recommended_places": [
    {
      "rank": 1,
      "name": "Cafe Tel Aviv",
      "google_rating": 4.8,
      "distance_meters": 150,
      "estimated_walk_minutes": 2,
      "address": "King George St 8"
    },
    {
      "rank": 2,
      "name": "Bistro Local",
      "google_rating": 4.7,
      "distance_meters": 250,
      "estimated_walk_minutes": 3,
      "address": "Dizengoff St 12"
    },
    {
      "rank": 3,
      "name": "Artisan Bakery",
      "google_rating": 4.6,
      "distance_meters": 400,
      "estimated_walk_minutes": 5,
      "address": "Frischerman St 5"
    }
  ]
}
```

---

## 4. Edge Cases
* **Fewer than 3 results:** If less than 3 places meet both constraints, return only the qualifying places alongside a `status_flag: "INSUFFICIENT_QUALIFYING_OPTIONS"`.
* **Zero results:** If no places meet the criteria, return an empty list with a recommendation to either expand the radius or lower the rating threshold.