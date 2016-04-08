# Simple - How to use? #

There are already 2 two main methods available in pipe class, `append(_$rules_)` and `parse(_$data_)`. First of all you create an instance.

```
$pipe = new pipe();
```

And then you append the rules:

```
$slicer = new rule(new slice('<div class="news-item">', '</div>', HTML, MULTI_ITEM));
$pipe->append($slicer);
```

HTML - Tells parser to keep aggregated data as Html. (Use PLAIN\_TEXT to strip html tags)
MULTI\_ITEM - Tells that `slice` rule need to be applied multiple times since there are `many` (not just one) `news-item`s on the page. (User SINGLE\_ITEM if there is only 1 new-item)

So in this example if you do:
```
$html = '
  <body>
    <div class="news-item"><h2>News Title 1</h2><span>Content 1</span></div>
    <div>Some garbage</div>
    <div class="news-item"><h2>News Title 2</h2><span>Content 2</span></div>
  </body>';

$result = $pipe->parse($html);
```

Then $result will be
```
array(
  '<h2>News Title 1</h2><span>Content 1</span>',
  '<h2>News Title 2</h2><span>Content 2</span>'
)
```

In a more complex situation (reality) you need the result to be more structured. Think of an associative array of news items.

You need to use 4th parameter of `rule` class ($crawlers) as you instantiate it:
$crawlers is an associative array of rules that tells parser to how to make $result to have a good structure. (Where exactly to find the `title` and where exactly to find `content`)

In fact $crawlers tells parser to run `title` and `link` rules after you found a news-item.

Geeks Note that these rules (two rule-types of css) have their own 4th parameter ($crawlers). Its a recursive world you know!
```

$crawlers = array(
   'title' => new rule(new css('h2'), PLAIN_TEXT, SINGLE_ITEM),
   'content' => new rule(new css('span'), PLAIN_TEXT, SINGLE_ITEM/*, $content_crawlers for geeks */)
);

$slicer = new rule(new slice('<div class="news-item">', '</div>', HTML, MULTI_ITEM, $crawlers));
$pipe->append($slicer);

// $html = ... (See example above)
// OR
// $html = file_get_contents('http://www.example.com/news/technology');

$result = $pipe->parse($html);
```

Now $result is more structured:
```
array(
  array('title' => 'News Title 1', 'content' => 'Content 1'),
  array('title' => 'News Title 2', 'content' => 'Content 2')
)
```

# Rule Types #

There are currently 3 rule-types available,

  1. `slice($from, $to)` - get data between two strings.
  1. `regexp($pattern)` - apply a regular expression match and return back-references.
  1. `css($selector)` - get content inside any css3 element which you select using css3 selectors.

# Nested rules #

As you can see in my google-search example we want every search result have an array of `title`, `link`, `description`, so i use nested rules you can see below:

```

	$pipe = new pipe();
		
	$pipe->append(new rule(new slice('<!--m-->', '<!--n-->'), HTML, MULTI_ITEM,
					array(
					'title'	=> new rule(new regexp('<!--m-->.*?<a .* onmousedown="return [^>]*>(.*?)</a></h3>.*'), PLAIN_TEXT, SINGLE_ITEM),
					'link'	 => new rule(new css('li.g div.vsc div.s div.f cite'), PLAIN_TEXT, SINGLE_ITEM),
					'summary'  => new rule(new css('div.s span.st'), PLAIN_TEXT, SINGLE_ITEM)
					)
				)
			);
			
	return $pipe->parse($html);

```