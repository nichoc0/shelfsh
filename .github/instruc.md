Okay, a fresh start for Page 2 with a focus on milestone timeliness and icons sounds like a clear and effective approach! This is definitely doable in Power BI.

Here's how you can achieve this, especially the icon part:

**1. Ensure Your Data is Structured Correctly (Power Query)**

For this to work well, you need a table where each row represents a specific milestone for a PCR and includes its Baseline and Actual dates. Let's call this your "Prepared Milestones Table". It should look something like this:

| PCR# | Milestone Name | Baseline Date | Actual Date |
| :--- | :------------- | :------------ | :---------- |
| P001 | RCA            | 2025-01-15    | 2025-01-14  |
| P001 | CB1            | 2025-03-01    | 2025-03-05  |
| P001 | Eng Release    | 2025-06-10    |             |
| P002 | RCA            | 2025-02-01    | 2025-02-01  |
| ...  | ...            | ...           | ...         |

*   **How to get this structure:**
    *   If your current "Milestones Table" has columns like `RCA Baseline`, `RCA Actual`, `CB1 Baseline`, `CB1 Actual`, etc., all on one row per `PCR#`:
        1.  In the Power Query Editor, select your "Milestones Table".
        2.  Select the `PCR#` column (and any other identifier columns you want to keep, like `Title`).
        3.  Go to the "Transform" tab and click "Unpivot Columns" -> "**Unpivot Other Columns**". This will give you columns like `PCR#`, `Attribute` (e.g., "RCA Baseline", "RCA Actual"), and `Value` (the date).
        4.  Now, you need to separate the "Milestone Name" (e.g., "RCA") from the "Date Type" (e.g., "Baseline", "Actual") in the `Attribute` column. You can do this by:
            *   Splitting the `Attribute` column by a delimiter (e.g., the last space, or if it's consistently "MilestoneName_Baseline", then by "_"). This might create two new columns: `Milestone Name Part` and `Date Type Part`.
        5.  With `PCR#`, `Milestone Name Part`, and `Date Type Part` selected, select the `Value` column (which contains the dates).
        6.  Go to the "Transform" tab and click "**Pivot Column**".
            *   For "Values Column", choose the column that has your dates (the original `Value` column).
            *   Under "Advanced options", for "Aggregate Value Function", choose "**Don't Aggregate**" (important!).
        7.  This should pivot your "Date Type Part" so you get separate columns for "Baseline" and "Actual" dates for each `PCR#` and `Milestone Name Part`. Rename columns as needed (e.g., `Milestone Name Part` to `Milestone Name`).
    *   Ensure `Baseline Date` and `Actual Date` columns are set to the **Date** data type.
*   Click "Close & Apply" in Power Query.

**2. Create a DAX Calculated Column for "On Time" Status**

This column will determine if a milestone was met on time and provide a value we can use for the icon.

*   In Power BI Desktop, go to the "Data" view.
*   Select your "Prepared Milestones Table".
*   Go to the "Table tools" ribbon and click "New column".
*   Enter the following DAX formula (you can name the column `OnTimeIconValue` or similar):

    ```dax
    // filepath: DAX Calculated Column in 'Prepared Milestones Table'
    OnTimeIconValue = 
    VAR ActualDate = 'Prepared Milestones Table'[Actual Date]
    VAR BaselineDate = 'Prepared Milestones Table'[Baseline Date]
    RETURN
    IF(
        ISBLANK(ActualDate) || ISBLANK(BaselineDate), 
        0, // Value for "Pending" or "Data Missing" (no icon or a neutral one)
        IF(
            ActualDate <= BaselineDate, 
            1, // Value for "On Time" (checkmark icon)
            2  // Value for "Late" (e.g., warning or cross icon)
        )
    )
    ```
    *   This formula assigns:
        *   `1` if Actual is on or before Baseline (On Time).
        *   `2` if Actual is after Baseline (Late).
        *   `0` if either date is blank (Pending/Data Missing).

**3. Build Your Visual on Page 2**

*   Go to the "Report" view and select Page 2.
*   Add a **Table** visual.
*   Drag the following fields into the "Columns" well of the table:
    *   From your "PCR HeatMAP" table (or wherever `PCR#` and `Title` are unique): `PCR#`, `Title` (you'll use slicers to filter this down to one PCR).
    *   From your "Prepared Milestones Table":
        *   `Milestone Name`
        *   `Baseline Date`
        *   `Actual Date`
        *   Your new DAX calculated column `OnTimeIconValue`

**4. Apply Conditional Formatting for Icons**

This is where you add the checkmarks.

*   Select your Table visual.
*   In the "Visualizations" pane, click the "Format your visual" icon (paintbrush).
*   Expand the "**Cell elements**" section.
*   In the "Apply settings to" dropdown, select your `OnTimeIconValue` column.
*   Turn the **Icons** toggle to "On".
*   Click the `fx` button (Conditional formatting) next to the Icons toggle.
*   In the "Icons - OnTimeIconValue" dialog:
    *   **Format style:** Rules
    *   **Based on field:** Should already be your `OnTimeIconValue` column.
    *   **Rules:**
        *   **Rule 1:** `If value` **is** `1` `number` `and` **is less than or equal to** `1` `number` `then` (select a **checkmark icon** from the dropdown).
        *   Click "**+ New rule**".
        *   **Rule 2:** `If value` **is** `2` `number` `and` **is less than or equal to** `2` `number` `then` (select a **warning/cross icon** for "Late").
        *   (Optional) Click "**+ New rule**".
        *   **Rule 3:** `If value` **is** `0` `number` `and` **is less than or equal to** `0` `number` `then` (select a neutral icon like a dash, or choose "No icon").
    *   **Icon layout:** You might want to choose "**Icon only**" if you don't want to see the numbers 0, 1, 2 in the column, just the icons. Or "Left of data" / "Right of data" if you want to see both.
    *   Click "OK".

**5. Add Slicers**

*   Add a Slicer visual to your page.
*   Drag `PCR#` (from your "PCR HeatMAP" table) into the Slicer's "Field" well. This will allow users to select a specific PCR to view its milestone status.

Now, when a user selects a `PCR#` from the slicer, the table will show its milestones, their baseline and actual dates, and an icon indicating if each completed milestone was on time, late, or is still pending.
