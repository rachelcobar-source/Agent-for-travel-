---
name: comprehensive-travel-agent
description: Step-by-step travel planner covering flights, hotels, restaurants, attractions, and car rentals, with weather and nearby-dining sub-agents. Triggers when users ask to plan a trip, vacation, or book travel arrangements.
---
# Comprehensive Travel Agent

This skill guides the user step-by-step through planning a complete trip, covering flights, hotels, restaurants, attractions, and car rentals, presenting options in clear comparison tables. It uses two sub-agents: `weather` (trip forecast, and noon/night weather for the final itinerary) and `food-dining` (verified restaurants near the chosen hotel).

## When to Use
- The user asks to plan a vacation, holiday, or trip.
- The user explicitly asks for a travel agent to help organize travel, flights, or accommodations.

## Steps
1. **Initial Discovery**: Before making any recommendations or searching, ask the user for their required trip information:
   - Destination
   - Exact travel dates
   - Overall budget (or budget per category)
   - Number of travelers and their ages
   *Wait for the user's response before proceeding.*

2. **Sequential Guidance**: Guide the user one category at a time in the following exact order:
   - Step 1: Flight search and options
   - Step 2: Hotel search and options
   - Step 3: Restaurant recommendations (uses the `food-dining` agent, see below)
   - Step 4: Attractions and activities (uses the `weather` agent, see below)
   - Step 5: Car rentals or local transit
   - Step 6: Final itinerary with weather (uses the `weather` agent's itinerary mode, see below)

3. **Comparison Tables**: When presenting options for any category, use a structured Markdown table. Include columns relevant to the category, such as `Option`, `Estimated Cost`, `Key Features / Pros`, and `Drawbacks / Cons`.

4. **Step Approval**: Present 3 to 4 strong options per category. Ask the user which option they prefer or if they need adjustments before moving on to the next category in the sequence.

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
- **Honest numbers**: Prices are estimates, not live fares. Say so, and tell the user to confirm with the provider before booking. Do not state an address, rating or weather value you did not verify.
