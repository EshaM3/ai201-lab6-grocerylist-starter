# Code Review Notes

Fill this in as you work through the milestones. Each section mirrors the structure of a real GitHub pull request review.

---

## PR #1 — Bulk Purchase (`pr1_bulk_purchase.py`)

### Summary
*What does this PR do? (1–2 sentences in your own words)*

>This creates a new feature where a user can mark all unpurchased items as purchased by a certain user under a certain grocery list. This saves time on having to mark each individual item as purchased during or after a long shopping trip.

### Issues

For each issue you find, note: where it is (file + function), what's wrong, and why it matters in production.

**Issue 1**
- Location: line 30
- What's wrong: not filtering by is_purchased
- Why it matters: 3 issues occur because of this: The purchased_at and purchased_by get overwritten by the current user and time for already purchased items, messing up the integrity of the data. It could make any user think that this user purchased more items at one time and that those items were not purchased earlier. Also, the returned item count becomes the total item count instead of the promised count of only recently purchased items by the user.
- Suggested fix: items = Item.query.filter_by(list_id=list_id, is_purchased=False).all()

**Issue 2**
- Location: lines 52-54
- What's wrong: After attempting to get user_id, null status of user_id is not verified before using it as a service call parameter.
- Why it matters: The purchased_by value can show up as 'null' if a proper user id is not given.
- Suggested fix: Add this line after line 52:
if not user_id:
    return jsonify({"error": "Missing required field: user_id"}), 400

<!-- **Issue 3** *(if found)*
- Location:
- What's wrong:
- Why it matters:
- Suggested fix: -->

### Questions for the Author
*Things you're uncertain about — design choices that could be intentional or bugs depending on intent.*

>N/A

### Verdict
- [ ] Approve — ship it
- [-] Request Changes — needs fixes before merging
- [ ] Comment — needs discussion before a verdict

**Rationale** *(1–2 sentences)*:

>2 issues are listed above that need to be corrected for proper functioning and handling of important edge cases.

---

## PR #2 — List Stats (`pr2_list_stats.py`)

### Summary
*What does this PR do? (1–2 sentences in your own words)*

>It provides helpful stats for a shopper by giving them quick details about the total items they have on the list, how much was purchased, how much is left, which aisles they have to go to for the remaining items, and how many items are in each aisle.

### Issues

**Issue 1**
- Location: lines 41-44
- What's wrong: The function is providing the category counts of the total items in the grocery list. We only want the category counts of remaining items.
- Why it matters: The user is using the stats to track what is left to purchase at different aisles. Not only would category counts of all items (including remaining items) not be a very useful stat, but it would also be quite confusing as it goes against what the function is promising to do.
- Suggested fix: Add a line under the for loop opening line like so:
for item in items:
    if not item.is_purchased:
        # rest of the for loop functionality

**Issue 2**
- Location: lines 62-63
- What's wrong: Error handling for missing list_id is missing.
- Why it matters: There is a lack of consistency in error handling with the current app. Usually, if list_id is a bad ID, we return a 404 error.
- Suggested fix: Add a try/except block around the current lines 62-63 as so:
try:
    stats = list_service.get_list_stats(list_id)
    return jsonify(stats), 200
except ValueError as e:
        return jsonify({"error": str(e)}), 404

<!-- **Issue 3** *(if found)*
- Location:
- What's wrong:
- Why it matters:
- Suggested fix: -->

### Questions for the Author
*A good code review often surfaces design questions, not just bugs. What would you want to clarify before approving?*

>N/A

### Verdict
- [ ] Approve — ship it
- [-] Request Changes — needs fixes before merging
- [ ] Comment — needs discussion before a verdict

**Rationale** *(1–2 sentences)*:

>2 issues are listed above that need to be corrected for a semantic error and a consistency issue.

---

## Reflection

*Answer after completing both reviews.*

**1.** Which issue was hardest to spot, and why?

>The first issue in PR2 (semantic error); I missed that detail in the instructions initially that said it only wanted categorized counts for *remaining* items.

**2.** Which issues do you think an LLM reviewer (like Claude reviewing its own code) would most likely miss? Why?

>I also think the semantic error is harder to catch at a glance than just missing error handling or extra filters (which seem to be faster checks for what is missing). A semantic error isn't just an extra line of protection that would be missing. It is related to a more specific request that the user made, which would be different from a more generic or broad implementation of a category-based item count.

**3.** One thing you'd add to a code review checklist for AI-generated backend code:

>Maintain idempotency
    >repeated calls should not alter already-purchased items again
    >AI code often forgets that bulk actions should be safe to re-run
