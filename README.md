#Faceted search with a color picker.

#Creating a color picker

It is highly important that an e-commerce store provides users with the option to apply filters while browsing collections. In this example, we will create a color picker such that a user can choose to display only a certain color of the product they searched for (e.g. "dress"). 

### Create your base HTML markup 

First, let's acquire the necessary Javascript libraries and basic markup. 

Download into the same directory 3 files: `jquery.ba-hashchange.min.js`, `jquery.swiftype.search.js` and `search.css` from the [Swiftype Search jQuery library](https://github.com/swiftype/swiftype-search-jquery).

In the same directory, create your own files `index.html` and `custom.css`.

In the following example of a base markup, we create search functionality through the Swiftype API. Note that the search results are contained in the element specified by the option `resultContainingElement`.

###Example: _index.html_  

    <!DOCTYPE html>
    <html>
      <head>
        <script type='text/javascript' src='https://ajax.googleapis.com/ajax/libs/jquery/1.11.1/jquery.min.js'></script>
        <script type='text/javascript' src='jquery.ba-hashchange.min.js'></script>
        <script type='text/javascript' src='jquery.swiftype.search.js'></script>
        <link type='text/css' rel='stylesheet' href='search.css' media='all' />
        <link type='text/css' rel='stylesheet' href='custom.css' media='all' />
        
        <div class="left">
            <div class="st-dynamic-facets">
            </div>
        </div>

        <script type='text/javascript'>

        $(function() {
            $('#st-search-input').swiftypeSearch({
              resultContainingElement: '#st-results-container',
              engineKey: 'VMnz-Ws16PuL8HyLPciN'
            });
        });
        
        </script>
      </head>
      <body>

        <div class="main">
            <h1>Gallivant</h1>
            <form>
              Search your site: <input type='text' id='st-search-input' class='st-search-input' />
            </form>
            <div id="st-results-container" class="st-result-listing"></div>
        </div>
      </body>
    </html>

###Example: _custom.css_

    .left {
        width: 128px;
        margin-left:20px;
        margin-top:45px;
        float:left;
    }

    .main {
        margin-left:178px;
    }

    .thumbnail{
        width:50px;
        height:60px;
        display:block;
    }

    .color {
        float: left;
        width: 20px;
        height: 20px;
        margin: 5px 5px 5px 5px;
    }

##Adding a color picker

###Example: _Adding faceted search_

          $('#st-search-input').swiftypeSearch({
              // code from previous steps
              facets: { 'page': ['color'] },
              postRenderFunction: bindControls,
              filters: readFilters,
           });

    var searchConfig = {
      facets: {},
    };

    var $facetContainer = $('.st-dynamic-facets');

    var readFilters = function() {
            return {
                'page': {color: window.searchConfig.facets['color']}
            }
        }

    var reloadResults = function() {
        $(window).hashchange();
    };

    var bindControls = function(data) {
        var resultInfo = data['info'],
        facets = '';

        $.each(resultInfo, function(documentType, typeInfo){
            $.each(typeInfo.facets, function(field, facetCounts) {
                facets += ['<div class="facet"><h3>', field, '</h3></div>'].join('')
                $.each(facetCounts, function(label, count) {
                    var status = "",
                    id = encodeURIComponent(label).toLowerCase();
                    if (window.searchConfig.facets[field] && window.searchConfig.facets[field].indexOf(label) > -1) {
                      status = ' checked="checked"'
                    }

                    facets += '<input type="checkbox"'  + ' style="display:none" ' + status + ' name="' + field + '" id="' + id + '" value="' + label +  '"> <label for="' + id + '">' + ' (' + count + ')' + '<div class="color" id="' + id + '"></div>' + '</label>' + '<br/><br/>';
                });
                facets += '<a href="#" class="clear-selection" data-name="' + field + '">Clear all</a>'
            });
            $facetContainer.html(facets);
        });

        $('.color').each(function(i, obj){
            $(obj)[0].style.backgroundColor = $(obj).attr('id');
        });
    };

    $facetContainer.on('click', 'input', function(e) {
        window.searchConfig.facets = {};
        $('.st-dynamic-facets input[type="checkbox"]').each(function(idx, obj) {
            var $checkbox = $(obj),
            facet = $checkbox.attr('name');
            if(!window.searchConfig.facets[facet]) {
              window.searchConfig.facets[facet] = [];
            }
            if($checkbox.is(':checked')) {
                window.searchConfig.facets[facet].push($checkbox.attr('value'));
            }
        });

        reloadResults();
    });

    $facetContainer.on('click', 'a.clear-selection', function(e) {
        e.preventDefault();
        var name = $(this).data('name');
        $('input[name=' + name + ']').attr('checked', false);
        window.searchConfig.facets[name] = [];

        reloadResults();
    });

Lets take this step-by-step:

We want to add the following options to our `swiftypeSearch` call.

           $('#st-search-input').swiftypeSearch({
              // code from previous steps
              facets: { 'page': ['color'] },
              postRenderFunction: bindControls,
              filters: readFilters,
           });

facets specifies DocumentType and attributes to give you a count of results in those attributes. Here we receive the count of each existing color.

postRenderFunction takes a function that is called after results are returned from Swiftype. It is responsible for rendering the facets on the page.

Next, we declare the `facets` object to store filter values. `$facetContainer` provides easier access for future reference. `readFilters` will return the facet settings of what to be filtered on. `reloadResults` serves to reload new search results after 

    var searchConfig = {
      facets: {},
    };

    var $facetContainer = $('.st-dynamic-facets');

    var readFilters = function() {
            return {
                'page': {color: window.searchConfig.facets['color']}
            }
        }

    var reloadResults = function() {
        $(window).hashchange();
    };

After the search has been made, bindControls is called to collect and render the faceted info.

    var bindControls = function(data) {
        var resultInfo = data['info'],
        facets = '';

        $.each(resultInfo, function(documentType, typeInfo){
            $.each(typeInfo.facets, function(field, facetCounts) {
                facets += ['<div class="facet"><h3>', field, '</h3></div>'].join('')
                $.each(facetCounts, function(label, count) {
                    var status = "",
                    id = encodeURIComponent(label).toLowerCase();
                    if (window.searchConfig.facets[field] && window.searchConfig.facets[field].indexOf(label) > -1) {
                      status = ' checked="checked"'
                    }

We iterate through the data for facet info. We only have one field in our case which is color. `facetCounts` includes information about all individual colors and their counts. We initialize the status of each color as being inactive as a facet setting with the empty string. The if-statement adds a checked tag to your HTML if a color is active in the filter for keeping track.

            facets += '<input type="checkbox"'  + ' style="display:none" ' + status + ' name="' + field + '" id="' + id + '" value="' + label +  '"> <label for="' + id + '">' + ' (' + count + ')' + '<div class="color" id="' + id + '"></div>' + '</label>' + '<br/><br/>';
                });
                facets += '<a href="#" class="clear-selection" data-name="' + field + '">Clear all</a>'
            });
            $facetContainer.html(facets);

We construct an `<input>`checkbox and `<label>` element for each color. We use the checkbox type for ease of implementation but hide it with the `style="display:none"` option. In place, we construct a colored box for the user to click on with a `<div>` element with `class="color"` as defined in `custom.css`.

When finished iterating through the collection of facets, we add a "Clear All" link and write the HTML contents to `$facetContainer`.

Next, we iterate through the colors and dynamically change the CSS color background for each color box. If CSS does not recognize the color value, it will default to white. 

        $('.color').each(function(i, obj){
            $(obj)[0].style.backgroundColor = $(obj).attr('id');
        });

Lastly, we create two event handlers to update the filter values in `searchConfig` if a color is clicked. We create an array for our colors if it doesn't yet exist. Then we add the color as a filter value if it has been clicked on by the user.

    $facetContainer.on('click', 'input', function(e) {
        window.searchConfig.facets = {};
        $('.st-dynamic-facets input[type="checkbox"]').each(function(idx, obj) {
            var $checkbox = $(obj),
            facet = $checkbox.attr('name');
            if(!window.searchConfig.facets[facet]) {
              window.searchConfig.facets[facet] = [];
            }
            if($checkbox.is(':checked')) {
                window.searchConfig.facets[facet].push($checkbox.attr('value'));
            }
        });

        reloadResults();
    });

    $facetContainer.on('click', 'a.clear-selection', function(e) {
        e.preventDefault();
        var name = $(this).data('name');
        $('input[name=' + name + ']').attr('checked', false);
        window.searchConfig.facets[name] = [];

        reloadResults();
    });
