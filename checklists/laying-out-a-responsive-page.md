# Laying out a responsive page

When: Starting a new page or section that has to work from a small phone up to a wide desktop screen.

How to use: Use this when you start a new page or section that has to work from a small phone up to a wide desktop screen. Answer the two questions first so you only see the checks for your case. Each check names what to do, and How to verify says what to look at and the sign it is right.

Before you start: Is anything on this page reused in slots of different widths? (any that apply) Yes, blocks are reused / None of these. Checks marked "only if" apply only to that answer.

## 1. Before you write layout CSS

You have the design or the content and are about to choose the page's structure.

Must:
- [ ] Add the viewport meta tag (source 1)
  Why: Set width to device-width so phones use their real width instead of a shrunk 980 pixel layout.
  How to verify: Open the page on a phone: it renders at phone width, not as a shrunk desktop, and **pinch zoom still works**.
- [ ] Write markup in reading order
  Why: Screen readers and the Tab key follow the source order, so write it in the order a reader needs it.
  How to verify: With styles turned off, the page reads header, navigation, then main content, from top to bottom.

Don't forget these too:
- [ ] Start from the narrowest layout
  Why: Write the single-column phone layout as the base, then add min-width queries as space opens up.
  How to verify: Styles outside any media query describe one readable column, and wider layouts only add to it.
- [ ] Sketch narrow, medium and wide
  Why: Decide which blocks sit side by side and which stack at each size before choosing grid or flex.
  How to verify: You have a sketch or note for each size that places every block of the page.
- [ ] Use grid for rows and columns (source 2)
  Why: Grid places blocks on rows and columns together, so the layout does not depend on each item's size.
  How to verify: Page regions and card grids use grid; no flex item carries a width set only to line up wrapped rows.
- [ ] Cap text at a readable measure (source 3)
  Why: Lines of about 45 to 75 characters read comfortably; set the text column's max width in ch, one character wide.
  How to verify: On the widest screen, count a line of body text: it holds **about 75 characters** or fewer.
- [ ] Choose a spacing scale up front (source 4)
  Why: Pick a few steps from one scale for gaps and padding, so spacing stays consistent across layouts.
  How to verify: Gaps and padding in the CSS use the scale's variables, not one-off pixel values.
- [ ] Decide which blocks query containers (source 5) (only if: Yes, blocks are reused)
  Why: Reusable blocks like cards adapt to the slot they sit in; page regions still follow the viewport.
  How to verify: Your notes name each reused block and the narrowest slot it will appear in.
- [ ] Set border-box sizing globally (source 6)
  Why: With border-box, padding and borders sit inside a box's width, so a 100 percent wide box with padding still fits its column.
  How to verify: In the base styles, every element and pseudo-element uses **box-sizing border-box**.
- [ ] Plan wide tables and charts
  Why: Decide with the designer whether each many-column table becomes cards, scrolls on its own, or drops columns on phones.
  How to verify: Every table and chart on the page has a written phone behavior agreed with the designer.

Stop if:
- If the design shows only one desktop width: ask the designer for phone and tablet layouts first

Move on when every block has a place at narrow, medium and wide sizes; base styles are the phone layout.

## 2. While building each section (every time)

You are writing the grid, flex and sizing rules for one section of the page.

Must:
- [ ] Set min-width 0 on shrinking items (source 7)
  Why: Flex and grid items stop shrinking at their content's width; min-width 0 lets them shrink past it.
  How to verify: Put a long URL in each column: it stays inside the column, and the page shows **no horizontal scrollbar**.
- [ ] Use max-width, not fixed widths
  Why: Let containers fill the available width up to a maximum, so they shrink on small screens.
  How to verify: Search the CSS for pixel widths on layout containers: each is a max-width or sits in a query where it fits.

Don't forget these too:
- [ ] Size columns with minmax and fr
  Why: A minmax minimum plus fr units, shares of the leftover space, lets columns grow and shrink between breakpoints.
  How to verify: Drag the window slowly: columns resize smoothly instead of jumping or overflowing.
- [ ] Let card grids wrap with auto-fit (source 8)
  Why: Use auto-fit with a minimum column width so the column count follows the space, with no media query.
  How to verify: Narrow the window: cards drop from several columns to one, none ever narrower than its minimum.
- [ ] Space items with gap (source 9)
  Why: Gap spaces grid and flex items without margins on the outer edges, so wrapped rows stay even.
  How to verify: When items wrap, row and column spacing match and no item carries an extra edge margin.
- [ ] Make images and media fluid
  Why: Cap images and video at their column's width and give them an aspect ratio, so they shrink in proportion.
  How to verify: At the narrowest phone width, no image, video or embed sticks out past its column or looks stretched.
- [ ] Scale type fluidly with clamp
  Why: Add a rem term to the clamp's middle value, not vw alone, so headings grow with the screen and still enlarge on zoom.
  How to verify: Zoom the browser in: headings get larger instead of staying stuck at a viewport-based size.
- [ ] Use svh for full-height sections (source 10)
  Why: On phones vh includes space under the browser bars; svh, small viewport height, fits the area with bars shown, dvh tracks them.
  How to verify: On a real phone with the address bar showing, a full-height section ends at the screen's bottom edge.
- [ ] Cap the page on wide screens
  Why: Center content in a max-width wrapper so lines and cards stop stretching beyond your widest design.
  How to verify: On a very wide window, content sits in a centered column and nothing spreads edge to edge.
- [ ] Wrap long words and names (source 11)
  Why: Let long words, URLs and names break with overflow-wrap, and avoid fixed heights on boxes that hold text.
  How to verify: Paste the longest real string into each label and heading: it wraps inside its box and nothing is clipped.
- [ ] Use logical properties for spacing (source 12)
  Why: Inline start and end, instead of left and right, flip with right-to-left text, so one layout serves both.
  How to verify: Set the page direction to right to left: the layout mirrors cleanly with no stray left-only margins.
- [ ] Let wide blocks scroll alone
  Why: Put wide tables and code blocks in a wrapper with horizontal overflow set to auto, so only that block scrolls, never the page.
  How to verify: At the narrowest phone width, the table or code block scrolls sideways while **the page itself** stays put.

Stop if:
- If text only fits with a fixed height: stop, remove the height and let the box grow

Move on when the section resizes smoothly with the window and never causes a horizontal scrollbar.

## 3. When you add a breakpoint (every time)

The layout stops working at some width and you are about to add a media or container query.

Must:
- [ ] Set breakpoints where content breaks (source 13)
  Why: Widen the window until a line gets too long or a row gets cramped, and break there, not at device widths.
  How to verify: Each breakpoint matches a width where the layout visibly broke, such as a text block growing past about 10 words per line.

Don't forget these too:
- [ ] Write breakpoints in em units (source 14)
  Why: Em-based media queries switch layouts when users enlarge default text in Chrome and Firefox; Safari only does with a minimum font size set.
  How to verify: Raise Chrome's default font size: the page switches to its narrower layout before text gets cramped.
- [ ] Reuse a few shared breakpoints
  Why: Keep shared widths as named variables or tokens so sections switch together; add one-offs only where a section needs them.
  How to verify: Most media queries use the shared values, and each one-off has a comment saying why.
- [ ] Query the container for reused blocks (source 15) (only if: Yes, blocks are reused)
  Why: Container queries switch a block's layout by its slot's width, so one card works in a sidebar and a main column.
  How to verify: Place the block in its narrowest and widest slot: each shows the layout that fits **its own width**.
- [ ] Keep visual order matching source (source 16)
  Why: Grid placement and the order property move blocks only on screen; change the markup if reading order must change.
  How to verify: Tab through each layout: focus moves top to bottom and left to right, never jumping back up.
- [ ] Keep content on small screens
  Why: Collapse or reorder content rather than hiding it; phone users need the same information.
  How to verify: Compare phone and desktop: every link, figure and paragraph exists in both, at most behind a toggle.
- [ ] Collapse nav when it overflows
  Why: Pick the collapse width from where the longest menu labels no longer fit on one line.
  How to verify: At every width, nav items sit on one line or behind the menu button, never in two ragged rows.

Stop if:
- If a block needs different breakpoints in each place it sits: switch it to container queries

Move on when the new breakpoint fixes its width, and no width just above or below it breaks.

## 4. Before you call it done

The page works in your own window; now test it the way real readers will see it.

Must:
- [ ] Drag through every width (source 17)
  Why: Drag DevTools' responsive viewport slowly from 320 pixels to your widest screen; layouts usually break between breakpoints, not at them.
  How to verify: At no width does a horizontal scrollbar appear or text overlap another block.
- [ ] Test reflow at high browser zoom
  Why: Zoom a desktop window until the page's CSS width is phone-sized, the accessibility reflow test; data tables and maps may scroll.
  How to verify: Everything reads in one column with only vertical scrolling, and no content is cut off.
- [ ] Enlarge the default text size
  Why: Change the browser or phone text size, not only page zoom, since many readers only change text size.
  How to verify: With large text, buttons and labels grow their boxes, and nothing is clipped or overlapped.

Don't forget these too:
- [ ] Limit sticky bars on short screens
  Why: Let headers and footers scroll away at short heights or large text, so they never fill the screen.
  How to verify: On a phone turned sideways with large text, sticky bars together take **well under half** the screen height.
- [ ] Test on a real phone (source 18)
  Why: Browser emulation misses the address bar, the on-screen keyboard and notches; one real phone catches them.
  How to verify: Rotate the phone and open the keyboard on a form: content stays reachable and nothing hides under a notch.
- [ ] Size tap targets for thumbs (source 19)
  Why: Aim for about 44 by 44 CSS pixels per tap target, with space between them, and more room near the screen's top and bottom.
  How to verify: On the phone layout, tap each link and button with a thumb: you hit the one you meant first time.
- [ ] Serve phone-sized images (source 20)
  Why: List files in srcset with width descriptors like small.jpg 480w, plus sizes, the width shown, so phones download the small file.
  How to verify: In the browser's network panel at phone width, each image loads a file **near its shown width**, not the desktop one.
- [ ] Check image crops at every width
  Why: Cropped images that fill their box can cut off faces and products as the box changes shape; set the focus point where they do.
  How to verify: Drag through the widths: faces and products stay whole in every cropped image.
- [ ] Test hover menus with touch
  Why: Touch screens have no hover, so keep hover-only extras behind a hover media query and give menus a tap to open.
  How to verify: On a phone, every menu and tooltip opens with a tap and **nothing needed** appears only on hover.
- [ ] Give form fields full-size text
  Why: iPhone Safari zooms in on a field whose text is below the default size, so give inputs the default text size or larger.
  How to verify: On an iPhone, tap each input: the page **does not zoom in**.

Stop if:
- If content scrolls sideways at the narrowest width: go to the Layout breaking at some screen sizes checklist
- If fixing large-text failures changes the design: take it to the designer before shipping

Move on when the page passes the width drag, high zoom, large text and real phone tests.

Not covered: Fixing a layout that already breaks, one component's API and content jumping while the page loads each have their own checklist. Full accessibility audits and cross-browser testing are covered separately.

Last reviewed: 2026-10-12

Sources:
1. MDN: Viewport concepts: https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/CSSOM_view/Viewport_concepts
2. MDN: Flexbox relationship to other layout methods: https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Flexible_box_layout/Relationship_with_other_layout_methods
3. web.dev: Learn Design, Typography: https://web.dev/learn/design/typography
4. Atlassian Design: Spacing foundations: https://atlassian.design/foundations/spacing
5. MDN: CSS container queries: https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Containment/Container_queries
6. paulirish.com: https://www.paulirish.com/2012/box-sizing-border-box-ftw
7. developer.mozilla.org: https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/min-width
8. CSS-Tricks: Auto-sizing columns, auto-fill vs auto-fit: https://css-tricks.com/auto-sizing-columns-css-grid-auto-fill-vs-auto-fit/
9. MDN: Defining gaps in CSS layouts: https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Gaps/Defining_gaps
10. web.dev Learn CSS: Sizing, viewport units: https://web.dev/learn/css/sizing/
11. MDN: Wrapping and breaking text: https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Text/Wrapping_breaking_text
12. MDN: CSS logical properties, basic concepts: https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Logical_properties_and_values/Basic_concepts
13. web.dev: Responsive web design basics: https://web.dev/articles/responsive-web-design-basics
14. Keith J Grant: px vs em in media queries: https://keithjgrant.com/posts/2023/05/px-vs-em-in-media-queries/
15. MDN: CSS container queries: https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_containment/Container_queries
16. Chrome for Developers: reading order in CSS: https://developer.chrome.com/blog/reading-order
17. developer.chrome.com: https://developer.chrome.com/docs/devtools/device-mode/
18. Chrome for Developers: Testing in other browsers: https://developer.chrome.com/docs/devtools/device-mode/testing-other-browsers
19. smashingmagazine.com: https://www.smashingmagazine.com/2023/04/accessible-tap-target-sizes-rage-taps-clicks/
20. web.dev: Serve responsive images: https://web.dev/articles/serve-responsive-images
21. web.dev: Baseline in action, container queries: https://web.dev/articles/baseline-in-action-container-queries

From CheatGrid: https://www.cheatgrid.com/checklists/frontend-developer/laying-out-a-responsive-page?utm_source=github&utm_medium=listing&utm_campaign=laying-out-a-responsive-page
