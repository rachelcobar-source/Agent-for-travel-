# Skill: Food & Dining Recommendation Sub-Agent

## Overview
This skill outlines the functional specification for a simplified **Food & Dining Sub-Agent**. The agent's sole purpose is to receive a hotel location from a Hotel Sub-Agent/Master Agent, scan the vicinity, and return up to **3 top dining recommendations** that satisfy two strict criteria. At least one of the recommendations must be **vegetarian-friendly**.

---

## 1. Core Constraints (Hard Criteria)
To be eligible for recommendation, a location **must** meet both parameters:
1. **Google Rating:** Strict minimum score above 4.5 (`rating > 4.5`).
2. **Proximity:** Walking distance less than 500 meters from the provided hotel coordinates (`distance < 500m`).

### Vegetarian Requirement
Every set of recommendations must include **at least one vegetarian-friendly place**. A place counts as vegetarian-friendly only if it is vegetarian/vegan, or clearly offers several vegetarian main dishes (a side salad is not enough). The vegetarian option must still meet both hard criteria above.

---

## 2. Agent Workflow

1. **Receive Context:** Read incoming JSON event containing the selected hotel's location/coordinates.
2. **Query & Filter:** Fetch nearby food places and eliminate any options with a Google rating $\le 4.5$ or distance $\ge 500\text{m}$.
3. **Check Vegetarian Fit:** Mark each valid place as `vegetarian_friendly: true/false`.
4. **Select Top 3:** Rank valid options using a combined metric of highest Google rating and shortest distance, selecting up to 3 best places.
5. **Guarantee a Vegetarian Option:** If none of the top 3 is vegetarian-friendly, replace the lowest-ranked place with the highest-ranked vegetarian-friendly place that passed the filter, then re-sort.
6. **Return Response:** Output structured JSON back to the calling agent.

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
      "address": "King George St 8",
      "vegetarian_friendly": false
    },
    {
      "rank": 2,
      "name": "Bistro Local",
      "google_rating": 4.7,
      "distance_meters": 250,
      "estimated_walk_minutes": 4,
      "address": "Dizengoff St 12",
      "vegetarian_friendly": false
    },
    {
      "rank": 3,
      "name": "Green Garden",
      "google_rating": 4.6,
      "distance_meters": 400,
      "estimated_walk_minutes": 5,
      "address": "Frischerman St 5",
      "vegetarian_friendly": true
    }
  ]
}
```

---

## 4. Edge Cases
* **Fewer than 3 results:** If less than 3 places meet both constraints, return only the qualifying places alongside a `status_flag: "INSUFFICIENT_QUALIFYING_OPTIONS"`.
* **Zero results:** If no places meet the criteria, return an empty list with a recommendation to either expand the radius or lower the rating threshold.
* **No vegetarian option:** If no qualifying place is vegetarian-friendly, return the regular top 3 and add `vegetarian_flag: "NO_VEGETARIAN_OPTION_FOUND"`. Never relax the rating or distance rules to find one.
