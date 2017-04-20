# ARIA treegrid example

[Jump to interactive version](https://cdn.rawgit.com/aleventhal/master/0.1.14/treegrid.html)

# Rationale

The ARIA treegrid has not gotten enough love. Docs the old specs provide recommendations for keyboard shortcuts that would conflict with browser/OS keys.

# Challenges and Approach

The main challenge is that the left/right arrow key could be used to collapse/expand or to move by column. This example attempts to resolve this conflict while remaining intuitive and discoverable. Essentially: left/right collapse/expand. If the current row is already expanded, the right arrow will begin cell navigation mode, and after that, arrows/home/end will navigate by cell. In addition, tab/shift+tab move between interactive cells in the current row. To exit cell navigation mode, move left past the first cell. Note that left/right/home/end are not supported when there is a focused text input cell, as it needs those keys.

* Ideally, screen readers would implement table navigation keys even in focus mode, launching from the current row or cell. NVDA does this in Firefox via Ctrl+Alt+Arrow, recognizing the current row as a starting position.

# Markup considerations
* roles used are treegrid, row and gridcell
* aria-level: used to set the current level of an item, 1, 2, 3, etc.
* aria-posinset, aria-setsize: used on a row to indicate the position of an item within it's local group, such as item 3 of 5. Unfortunately, this is not currently legal in the spec.
* aria-rowindex, aria-rowcount, aria-colindex, aria-colcount. Not currently used, as UA/AT can compute this for fully loaded table DOMs.
* aria-expanded (triststate): this attribute must be removed (not present) if the item cannot be expanded. For expandable items, the value is "true" or "false"
* aria-hidden: set to "true" for child items that are currently hidden because the parent is collapsed
* aria-owns: not currently used, awaiting discussion. Could be used for parents to identify their children, but it doesn't look like any screen readers actually use this so it seems to be a wasteful recommendation.
* aria-labelledby or aria-describedby for headers? Not currently used, awaiting discussion
* aria-activedescendant -- this example does not use, because it is not exposed to ATs in IE, and thus uses tabindex instead
* aria-readonly: be default, a grid/treegrid is editable, but these tend not to be. This is mentioned in the text of the ARIA spec for grid/treegrid but doesn't seem to have made its way into the AAM. This idea originated for grids, which are like spreadsheets, where most cells are probably editable. It may not make sense for a treegrid, but this is the legacy. Bottom line: if you don't want "editable" read for every cell in some browser-screen reader combinations, you'll need to put aria-readonly="true" on the appropriate role="gridcell" elements or on the grid/treegrid itself.
* aria-selected: used on row in the case of a multiselectable treegrid where each row begins with a checkbox. Must be set to false or true so that it is clear that row is selectable.
* tabindex is set in the JS, as per the usual roving tabindex methodology. Specifically, we use tabindex="0" for the current item so that it is the subitem that gets focused if user tabs out and back in, and tabindex="-1" for all items where we want click-to-focus behavior enabled.

# Observations with screen readers

<table>
<thead>
  <tr>
    <th></th>
    <th>Chrome (Canary currently required)</th>
    <th>Firefox</th>
    <th>IE11</th>
    <th>Safari</th>
  </tr>
</thead>
<tbody>
<tr>
<th>Chomevox</th>
<td>
<li>Row nav: Previously did not read name, should be fixed in Canary, needs test
<li>Cell nav: Does not read name on cells even if aria-label or aria-labelledby are set</td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<th>JAWS</th>
<td>
  <li>Row navigation: requires Chrome Canary otherwise won't read name
  <li>Levels not reported, item n of m not reported
  <li>Collapse/expand, nothing reported
</td>
<td>
  <li>Row navigation: levels not reported,
  <li>Column navigation: no issues found
  <li>Other: when collapsing/expanding: new collapsed/expanded state not spoken, says "row unselected", but nothing useful
</td>
<td>Need to use Insert+Z to turn virtual cursor off. Does not report level or item n of m.</td>
<td></td>
</tr>
<tr>
<th>NVDA</th>
<td>
  <li>Row navigation: requires Chrome Canary otherwise won't ready name
  <li>Column navigation: Does not read column headers
</td>
<td>
  <li>Row navigation: levels read correctly, all seems good
  <li>Column navigation: cells are announced as "selected" but this seems redundant since it's not an aria-multiselectable treegrid
  <li>Has very cool feature where Ctrl+alt arrow can navigate cells of table from current position, without
      additional help from the ARIA widget. If all screen readers did this then the widget author would not need
      to implement the cell navigation
<td>
  <li>Column navigation: unlike in Firefox, NVDA does not read column labels but instead reads the column number
  <li>Row navigation: row numbers are reported, unlike with NVDA and Firefox. Unfortunately both row 2 and 3 are reported as row 2 :/
  <li>The ctrl+alt+arrow feature that is available in Firefox does not work in IE
</tr>
<tr>
<th>VoiceOver</th>
<td>
  <li>First focus: calls it a listbox
  <li>Column navigation: reads each item twice
  <li>Row navigation: does not read expanded state, completely skips reading second column for second row
<td></td>
<td></td>
<td>Row navigation: doesn't let us do this, reads cells no matter what</td>
</tr>
</tbody>
</table>
