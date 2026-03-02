# OVERVIEW
The purpose of this Tableau extension is to compute a custom axis range for a viz and store it in a parameter.

Upon creating an instance of this extension, the creator selects a free-form numeric parameter (float or int), along with one or more fields of the same type from one or more worksheets. The extension retrieves the maximum value of that field from each of those worksheets, takes the maximum of those values, and stores it in the chosen parameter.

The extension then listens for any update to any *other* parameter, and recomputes whenever one is detected.

Its visible representation is a plain white box; it is assumed the creator will make it invisible.