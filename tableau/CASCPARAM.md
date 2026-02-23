# OVERVIEW
The purpose of this extension is to let the user populate parameter values from a hierarchy, using cascading drop-down menus. Each selection cascades to the next menu in the hierarchy. We require that the data forms a proper hierarchy, i.e., each item in a branch must correspond to exactly one item in the previous branch.

Upon creating an instance of this extension, the creator selects two or more fields from a data source, and specifies an ordering for those fields.
- The field can be a formula, or it can be straight from the data source.
- Upon loading, validate that the fields form a proper hierarchy. For each possible value of field #N+1, that value must occur with *exactly* one value of field #N. If this is not the case, raise an error and flag the duplicates.
- Upon loading, validate that the value "(All)" does not exist for any of the chosen fields.

The creator then matches each field to a free-form string parameter.

You *may* apply any or all of these requirements if it makes the code simpler:
- The creator must select a worksheet with those fields present; the hierarchy values will be loaded from that sheet on workbook load.
- The fields must be string-valued.
- The fields must be discrete or dimensions.

Here are two examples of how the menus should behave, followed by precise implementation logic.

# EXAMPLE #1

| Country | State | City |
| --- | --- | --- |
| United States | California | Los Angeles
| United States | California | San Francisco
| United States | Texas | Houston
| United States | Texas | Dallas
| United States | Texas | San Antonio
| United States | Washington | Seattle
| Canada | Quebec | Montreal

- The creator selects "Country", "State", and "City" as the fields and maps them to parameters "Pick Country", "Pick State", and "Pick City".
    - Initial drop-down options:
        - **Country**: "(All)", "Canada", "United States"
        - **State**: "(All)", "California", "Quebec", "Texas", "Washington"
        - **City**: "(All)", "Dallas", "Houston", "Los Angeles", "Montreal", "San Antonio", "San Francisco", "Seattle"
    - Initial selections/parameter values:
        - **Pick Country**: "(All)"
        - **Pick State**: "(All)"
        - **Pick City**: "(All)"
- The user selects "United States" from the first drop-down.
    - Drop-down options:
        - **Country**: "(All)", "Canada", "United States"
        - **State**: "(All)", "California", "Texas", "Washington"
        - **City**: "(All)", "Dallas", "Houston", "Los Angeles", "San Antonio", "San Francisco", "Seattle"
    - Selections/parameter values:
        - **Pick Country**: "United States"
        - **Pick State**: "(All)"
        - **Pick City**: "(All)"
- The user selects "California" from the second drop-down.
    - Drop-down options:
        - **Country**: "(All)", "Canada", "United States"
        - **State**: "(All)", "California", "Texas", "Washington"
        - **City**: "(All)", "Los Angeles", "San Francisco"
    - Selections/parameter values:
        - **Pick Country**: "United States"
        - **Pick State**: "California"
        - **Pick City**: "(All)"
- The user selects "Los Angeles" from the third drop-down.
    - Drop-down options:
        - **Country**: "(All)", "Canada", "United States"
        - **State**: "(All)", "California", "Texas", "Washington"
        - **City**: "(All)", "Los Angeles", "San Francisco"
    - Selections/parameter values:
        - **Pick Country**: "United States"
        - **Pick State**: "California"
        - **Pick City**: "Los Angeles"
- The user selects "Washington" from the second drop-down.
    - Drop-down options:
        - **Country**: "(All)", "Canada", "United States"
        - **State**: "(All)", "California", "Texas", "Washington"
        - **City**: "Seattle"
    - Selections/parameter values:
        - **Pick Country**: "United States"
        - **Pick State**: "Washington"
        - **Pick City**: "Seattle"
- The user selects "Canada" from the first drop-down.
    - Drop-down options:
        - **Country**: "(All)", "Canada", "United States"
        - **State**: "Quebec"
        - **City**: "Montreal"
    - Selections/parameter values:
        - **Pick Country**: "Canada"
        - **Pick State**: "Quebec"
        - **Pick City**: "Montreal"
- The user selects "(All)" from the first drop-down.
    - Drop-down options:
        - **Country**: "(All)", "Canada", "United States"
        - **State**: "(All)", "California", "Quebec", "Texas", "Washington"
        - **City**: "(All)", "Dallas", "Houston", "Los Angeles", "Montreal", "San Antonio", "San Francisco", "Seattle"
    - Selections/parameter values:
        - **Pick Country**: "(All)"
        - **Pick State**: "(All)"
        - **Pick City**: "(All)"
- The user selects "San Antonio" from the third drop-down.
    - Drop-down options:
        - **Country**: "(All)", "Canada", "United States"
        - **State**: "(All)", "California", "Texas", "Washington"
        - **City**: "(All)", "Dallas", "Houston", "San Antonio"
    - Selections/parameter values:
        - **Pick Country**: "United States"
        - **Pick State**: "Texas"
        - **Pick City**: "San Antonio"

# EXAMPLE #2
| Country | State | City |
| --- | --- | --- |
| United States | California | Los Angeles
| United States | California | San Francisco
| United States | Maine | Portland
| United States | Oregon | Portland
| United States | Texas | Houston
| United States | Texas | Dallas
| United States | Texas | San Antonio
| United States | Washington | Seattle

This raises an error: `Invalid hierarchy: City "Portland" occurs with State "Maine" and State "Oregon".`

# IMPLEMENTATION LOGIC

1. Generate one drop-down menu per field, lined up horizontally from left to right.

2. Compute the contents of each drop-down:
    - Drop-down #1 contains all possible values of field #1 in the database.
        - List them in alphabetical order.
        - If this results in at least two options, add the option "(All)" and put it first in the list.
        - Select the first option in the list.
    - Drop-down #N contains all values of field #N which occur with the selected value of field #N-1 from drop-down #N-1.
        - If the selected value of drop-down #N-1 is "(All)", then match to drop-down #N-2 instead. If that is "(All)", match to #N-3, et cetera. If all previous drop-downs are set to "(All)", then use all values of field #N.
        - List them in alphabetical order.
        - If this results in at least two options, add the option "(All)" and put it first in the list.
        - Select the first option in the list.

3. Whenever the user changes the selected value of drop-down #N:
    - A) For drop-down #N+1, recompute its contents, then select the first value in the updated list. Then repeat for drop-down #N+2, then #N+3, et cetera.
    - If the selected value of drop-down #N is not "(All)":
        - B) Select the value from drop-down #1 which occurs with the newly selected value of field #N.
        - C) Recompute the contents of drop-down #2, then select the value which occurs with the newly selected value of field #N.
        - D) Repeat for drop-down #3, #4, etc., up to #N-1.
        - E) Recompute the contents of drop-down #N, but don't change the selected value.

4. Whenever the selected value of a drop-down changes (either because the user made a selection or because of rule 3), set the corresponding parameter to the newly selected value.

5. Each copy of the extension monitors its corresponding parameters for changes. If it detects a change to one or more of those parameters, and the new value does not match the current selection in the drop-down:
  - Choose the deepest-level change whose value is not "(All)" (i.e., if there are changes in parameters #2, #3, and #5, take the change in #5). Apply that change to the appropriate drop-down. Then apply rule 3 normally.
  - If all of the changed parameters have become "(All)", instead choose the shallowest-level change. Apply it and apply rule 3.
  - This ensures that multiple copies of the same hierarchy on different dashboards remain in sync.
  - Fallback if the listener system proves unreliable: Poll the current parameter values once per second and look for mismatches. If any are found, apply as above.

6. The extension configuration dialog includes a button to manually refresh hierarchy data. Otherwise the data is assumed to be static after workbook load.

# HOSTING
This extension is published in a public GitHub repo. The organization is third-coast-federal and the repo is tcf-public.