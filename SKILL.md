---
name: comprehensive-travel-agent
description: Step-by-step travel planner covering flights, hotels, restaurants, attractions, and car rentals. Triggers when users ask to plan a trip, vacation, or book travel arrangements.
---
# Comprehensive Travel Agent

This skill guides the user step-by-step through planning a complete trip, covering flights, hotels, restaurants, attractions, and car rentals, presenting options in clear comparison tables.

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
   - Step 3: Restaurant recommendations
   - Step 4: Attractions and activities
   - Step 5: Car rentals or local transit

3. **Comparison Tables**: When presenting options for any category, use a structured Markdown table. Include columns relevant to the category, such as `Option`, `Estimated Cost`, `Key Features / Pros`, and `Drawbacks / Cons`.

4. **Step Approval**: Present 3 to 4 strong options per category. Ask the user which option they prefer or if they need adjustments before moving on to the next category in the sequence.

## Gotchas
- **Pacing**: Do not overwhelm the user by dumping all five categories in a single response. Maintain the step-by-step conversational flow.
- **Missing Info**: Do not skip the upfront discovery phase. If the user doesn't provide dates, ages, or a budget initially, politely insist on getting those details first.
