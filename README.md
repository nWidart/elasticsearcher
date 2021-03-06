# elasticsearcher

[![Circle CI](https://circleci.com/gh/madewithlove/elasticsearcher.svg?style=svg)](https://circleci.com/gh/madewithlove/elasticsearcher)

This agnostic package is a lightweight wrapper on top of the [Elasticsearch PHP client](http://www.elastic.co/guide/en/elasticsearch/client/php-api/current/index.html).
Its main goal is to allow for easier structuring of queries and indices in your application. It does not want to hide or replace
functionality of the Elasticsearch PHP client.

## Installation

Installation of the latest version is easy via [composer](https://getcomposer.org/):

```
composer require madewithlove/elasticsearcher
```
## Features

### Query class

Structure queries inside a class for clearer oversight in your application.

```php
class MoviesFrom2014Query extends AbstractQuery
{
	public function setup()
	{
		$this->searchIn('movies', 'movies');

		// Full notation
		$body = [
			'query' => [
				'filtered' => [
					'filter' => [
						'term' => ['year' => 2014]
					]
				]
			]
		];
		$this->setBody($body);

		// Short (dotted) notation
		$this->set('query.filtered.filter.term.year', 2014);
	}
}

// Usage
$query = new MoviesFrom2014Query($this->getElasticSearcher());
$query->run();
```

### Query with custom/re-usable fragments

Move re-occuring or complex fragments of your query or index to a separate class.

```php
class MoviesFrom2014Query extends AbstractQuery
{
	public function setup()
	{
		$this->searchIn('movies', 'movies');

		$this->set('query.filtered.filter', [new YearFilter(2014)]);
	}
}
```

### Query with custom result parsing

Perform actions on the response from Elasticsearch before the Query returns the results. It can be used for converting
the Elasticsearch documents into models/entities from your ORM. Re-use it in multiple queries.

```php
class MoviesFrom2014Query extends AbstractQuery
{
	public function setup()
	{
		$this->searchIn('movies', 'movies');
		$this->parseResultsWith(new MoviesResultParser());

		$body = array(...);

		$this->setBody($body);
	}
}

// Usage
$query = new MoviesFrom2014Query($this->getElasticSearcher());
$result = $query->run();
foreach ($result->getResults() as $movie) {
	var_dump($movie->title, $movie->id, $movie->year);
}
```

### Indices management

```php
$searcher->indicesManager()->exists('listings');
$searcher->indicesManager()->existsType('suggestions', 'movies');
$searcher->indicesManager()->create('suggestions');
$searcher->indicesManager()->update('suggestions');
$searcher->indicesManager()->delete('suggestions');
$searcher->indicesManager()->deleteType('suggestions', 'movies');
```

### Documents management

```php
$manager->index('suggestions', 'movies', $data);
$manager->bulkIndex('suggestions', 'movies', [$data, $data, $data]);
$manager->update('suggestions', 'movies', 123, ['name' => 'Fight Club 2014']);
$manager->updateOrIndex('suggestions', 'movies', 123, ['name' => 'Fight Club 2014']);
$manager->delete('suggestions', 'movies', 123);
$manager->exists('suggestions', 'movies', 123);
$manager->get('suggestions', 'movies', 123);
```

### Access to Elasticsearch client

The package does not and will not try to implement everything from the Elasticsearch client. Access to the client is
always possible.

```php
$client = $searcher->getClient();
```

# Usage

```php
use ElasticSearcher\Environment;
use ElasticSearcher\ElasticSearcher;

$env = new Environment(
  ['hosts' => ['localhost:9200']]
);
$searcher = new ElasticSearcher($env);
```

More usage in the [examples](./examples) and documentation.

# Documentation

* [Index management](./docs/index-management.md)
* [Document management](./docs/document-management.md)
* [Query building (search)](./docs/query-building.md)
* [Result parsing (after search)](./docs/result-parsing.md)
* [Re-useable fragments](./docs/re-useable-fragments.md)
