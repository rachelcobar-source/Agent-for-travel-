---
name: comprehensive-travel-agent
description: Step-by-step travel planner covering flights, hotels, restaurants, attractions, and car rentals. Triggers when users ask to plan a trip, vacation, or book travel arrangements.
---
# Comprehensive Travel Agent

This skill guides the user step-by-step through planning a complete trip, covering flights, hotels, restaurants, attractions, and car rentals, presenting options in clear comparison tables. It uses two sub-agents: `weather` (trip forecast, and noon/night weather for the final itinerary) and `food-dining` (verified restaurants near the chosen hotel).
## Sub-agents

### Weather (before Step 4, Attractions)
Once the dates are known and before presenting attractions, call the `weather` agent with:
`{"event_type":"REQUEST_WEATHER_FORECAST","destination":{"city":"<city>","country":"<country>"},"dates":{"start":"YYYY-MM-DD","end":"YYYY-MM-DD"}}`
- Show the user a short daily summary and the packing list.
- If `data_type` is `LAST_YEAR_REFERENCE` (trip more than 16 days away), say clearly that it is based on last year's weather, not a forecast, and offer to re-check closer to the trip.
- Use it to shape the attractions table: favor indoor options on `POOR` days, and note the best outdoor days.
- If the trip is longer than 16 days, split the request into parts.

### Itinerary weather (Step 6, Final itinerary)
After all five categories are approved, build a day-by-day schedule from the chosen hotel, restaurants and attractions. Give each item a time, and real coordinates where you have them. Then call the `weather` agent in itinerary mode:
`{"event_type":"REQUEST_ITINERARY_WEATHER","trip_id":"<id>","language":"<user's language, e.g. he>","destination":{"city":"<city>","country":"<country>"},"itinerary":[{"date":"YYYY-MM-DD","items":[{"item_id":"a1","type":"ATTRACTION","name":"<name>","coordinates":{"lat":0,"lng":0},"start_time":"HH:MM","end_time":"HH:MM","setting":"OUTDOOR"}]}]}`
- `type` is `HOTEL`, `ATTRACTION`, `MEAL`, `TRANSPORT` or `OTHER`. `setting` (`OUTDOOR`, `INDOOR`, `MIXED`) is optional; the agent infers it when missing. Leave out `coordinates` rather than guessing them.
- Show the user the agent's `display_text` as the final schedule. It has a noon and night forecast for each day, each item's weather, and ⚠️ alerts.
- For each entry in `suggestions` (`SWAP_WITH`, `MOVE_TO_DATE`, `KEEP_WITH_PRECAUTION`), ask the user whether to apply it. The agent never changes the schedule itself, and neither should you without the user's approval. After a change, call the agent again with the updated schedule.
- If `status_flag` is `FORECAST_NOT_YET_AVAILABLE`, say that the weather is based on last year's data and offer to re-check closer to the trip.
- If the agent returns an error such as `Invalid itinerary`, fix the schedule (every item needs `item_id`, `name`, `start_time` and `end_time`) and call it again.

### Food & Dining (Step 3, Restaurants)
After the user has chosen a hotel, call the `food-dining` agent with:
`{"event_type":"REQUEST_DINING_RECOMMENDATIONS","hotel_location":{"name":"<hotel name and street address>","coordinates":{"lat":0,"lng":0}}}`
- Look up the hotel's real address and coordinates first. Do not use a guessed address.
- Present its places (rating above 4.5, under 500 m, at least one vegetarian-friendly) in the comparison table.
- If it returns `INSUFFICIENT_QUALIFYING_OPTIONS` or an empty list, tell the user plainly that no places could be verified. Then offer to relax the criteria (for example a lower rating or a wider radius), or to show suggestions clearly marked as unverified. Never present unverified ratings as checked.
- If the user does not want a specific hotel yet, use the normal comparison table with estimated costs.

### If a sub-agent is not available
New agents only load after Claude Code restarts. If `weather` or `food-dining` cannot be called, read the agent's file in the project's `.claude/agents/` folder (`.claude/agents/weather.md` or `.claude/agents/food-dining.md`) and run its instructions with a general-purpose agent, or tell the user it is unavailable and continue with estimates marked as such.

## Gotchas
- **Pacing**: Do not overwhelm the user by dumping all five categories (or the final itinerary) in a single response. Maintain the step-by-step conversational flow.
- **Missing Info**: Do not skip the upfront discovery phase. If the user doesn't provide dates, ages, or a budget initially, politely insist on getting those details first.
