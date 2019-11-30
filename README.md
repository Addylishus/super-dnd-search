# super-dnd-search
A search engine for DnD 5e.
var compendium = {};
var url = 'http://dl.dropboxusercontent.com/s/skhhsn2qi3ngcmx/data.json?dl=1';
jQuery(document).ready(function($){
  if(sessionStorage.getItem("compendium") == null || sessionStorage.getItem("compendium") == "undefined"){
    $.when(
      // $.ajax({
      //   type: 'GET',
      //   cache: true,
      //   // url: 'https://jsonp.afeld.me/?url=http://dl.dropboxusercontent.com/s/skhhsn2qi3ngcmx/data.json&raw=1',
      //   url: 'data.json',
      //   dataType: 'json',
      // })
      $.get('data.json'),
      $.get('rules.json')
    ).then(function(data, rules){
      //cache JSON for faster load times
      var c;
      c = $.extend(c, data[0], rules[0]);
      try{
        sessionStorage.setItem("compendium", JSON.stringify(c));
      } catch(e) {
        console.log("Couldn't cache the JSON file");
      }
      //assign JSON object
      init(c);
    }).fail(function(){
      console.log("failure loading data")
    });
  }else{ //pull from local cache
    init(JSON.parse(sessionStorage.getItem("compendium")));
  }
  $('.typeahead').bind("typeahead:active", function(ev){
    $('.typeahead').typeahead('val', '');
  });
  $('.typeahead').bind('typeahead:select', function(ev, suggestion) {
    var sp = getInfo(suggestion);
    createCard(sp);
  });
  populateChangelog();
});
function init(data){
  compendium = data;
  setupTA();
  populateBrowseMenus();
  //grab query params from URL, if someone linked you to a thing
  var q = decodeURI(location.search.substring(3)); // format: ?q=  - this is stripped out
  if(q != ""){
    q=q.toTitleCase();
    q=q.replace(/+/g, '%20');
    q=q.replace(/\+/g, '%20');
    $('.typeahead').typeahead('val', q);
    createCard(getInfo(q));
  }
