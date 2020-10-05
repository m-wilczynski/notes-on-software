[D3.js](/frameworks-and-libraries/d3js)
# Thinking in selections

D3.js offers an interesting, declarative way for creating data-driven documents in JavaScript.
Core of such approach are **selections**.

### Selection

Selection is a way of picking existing (or "soon to exist") elements in DOM that forms a grouping.
To start on root level use ```d3.select```; to select (further) on already existing selection use ```selection.select``` etc.

There are two ways of selecting things in D3.js API:
- **```select```** - will select first (or the only) matching element (in document order) from selection it operates on; </br>
 example: ```d3.selectAll('h2').select('div')``` will select **every 1st ```div``` descendant in every ```h2``` element** in document; </br> 
 since **select** chooses only first element, it will flatten the hierarchy and pass the associated data down to the selected children nodes</br>
 *TODO: Visualize*

- **```selectAll```** - will select all matching descendant element (in document order) from selection it operates on; </br>
 example: ```d3.selectAll('h2').selectAll('div')``` will select **all ```div``` descendant in every ```h2``` element** in document; </br>
 **selectAll** preserves previous grouping and creates selection hierarchy (often described as *'nested selection'*), therefore data is not passed down to children nodes; </br> 
 we can however assign data to such selection using enter, update and exit selections (operators?) and tell D3.js how to react to data changes (ie. add, update or remove elements, that visualize data in document)</br>
 *TODO: Visualize*

### Binding data to selections

 *TODO: enter, update and exit with ```data```, ```enter``` and ```exit``` </br>*
 *TODO: key function as a second param to ```data(values, keyFunc)```*

 #### References
- Mike Bostock, [How Selections Work](https://bost.ocks.org/mike/selection/)
- Mike Bostock, [Nested Selections](https://bost.ocks.org/mike/nest/)
- Mike Bostock, [Thinking with Joins](https://bost.ocks.org/mike/join/)