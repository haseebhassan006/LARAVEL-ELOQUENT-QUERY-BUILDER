# LARAVEL-ELOQUENT-QUERY-BUILDER

## Retrieving Models

> The model's **all** method will retrieve all of the records from the model's associated database table:
use App\Models\Flight;

`foreach (Flight::all() as $flight) {
    echo $flight->name;
}`

## Building Queries

>  Each Eloquent model serves as a query builder, you may add additional constraints to queries and then invoke the get method to retrieve the results:

`$flights = Flight::where('active', 1)
               ->orderBy('name')
               ->take(10)
               ->get();`
               
 ## Refreshing Models
 
 > If you already have an instance of an Eloquent model that was retrieved from the database, you can "refresh" the model using the fresh and refresh methods.
 
 ```
 $flight = Flight::where('number', 'FR 900')->first();
 $freshFlight = $flight->fresh();
  ```
 ##  Collections
 
 >The Eloquent Collection class extends Laravel's base Illuminate\Support\Collection class, which provides a variety of helpful methods for interacting with data collections.
 
 ```
 $flights = Flight::where('destination', 'Paris')->get();

$flights = $flights->reject(function ($flight) {
    return $flight->cancelled;
});
```
> Since all of Laravel's collections implement PHP's iterable interfaces, you may loop over collections as if they were an array:

```
foreach ($flights as $flight) {
    echo $flight->name;
}
```
## Chunking Results

>The chunk method will retrieve a subset of Eloquent models, passing them to a closure for processing. Since only the current chunk of Eloquent models is retrieved at a time, the chunk method will provide significantly reduced memory usage when working with a large amount of models:

```
use App\Models\Flight;

Flight::chunk(200, function ($flights) {
    foreach ($flights as $flight) {
        //
    }
});
```
>The first argument passed to the chunk method is the number of records you wish to receive per "chunk". The closure passed as the second argument will be invoked for each chunk that is retrieved from the database.

> If you are filtering the results of the chunk method based on a column that you will also be updating while iterating over the results, you should use the chunkById method.
```
Flight::where('departed', true)
        ->chunkById(200, function ($flights) {
            $flights->each->update(['departed' => false]);
        }, $column = 'id');
```

## Cursors
> the cursor method may be used to significantly reduce your application's memory consumption when iterating through tens of thousands of Eloquent model records.

The cursor method will only execute a single database query; however, the individual Eloquent models will not be hydrated until they are actually iterated over. Therefore, only one Eloquent model is kept in memory at any given time while iterating over the cursor.
```
use App\Models\Flight;

foreach (Flight::where('destination', 'Zurich')->cursor() as $flight) {
    //
}
```
>The cursor returns an Illuminate\Support\LazyCollection instance

```
use App\Models\User;

$users = User::cursor()->filter(function ($user) {
    return $user->id > 500;
});

foreach ($users as $user) {
    echo $user->id;
}
```
## Advanced Subqueries
### Subquery Selects
>For example, let's imagine that we have a table of flight destinations and a table of flights to destinations. The flights table contains an arrived_at column which indicates when the flight arrived at the destination.
Using the subquery functionality available to the query builder's select and addSelect methods, we can select all of the destinations and the name of the flight that most recently arrived at that destination using a single query:
```
use App\Models\Destination;
use App\Models\Flight;

return Destination::addSelect(['last_flight' => Flight::select('name')
    ->whereColumn('destination_id', 'destinations.id')
    ->orderByDesc('arrived_at')
    ->limit(1)
])->get();

```
### Subquery Ordering
```
return Destination::orderByDesc(
    Flight::select('arrived_at')
        ->whereColumn('destination_id', 'destinations.id')
        ->orderByDesc('arrived_at')
        ->limit(1)
)->get();
```
## Retrieving Single Models / Aggregates

>In addition to retrieving all of the records matching a given query, you may also retrieve single records using the find, first, or firstWhere methods. 

```
use App\Models\Flight;

// Retrieve a model by its primary key...
$flight = Flight::find(1);

// Retrieve the first model matching the query constraints...
$flight = Flight::where('active', 1)->first();

// Alternative to retrieving the first model matching the query constraints...
$flight = Flight::firstWhere('active', 1);
```
>The firstOr method will return the first result matching the query or, if no results are found, execute the given closure. The value returned by the closure will be considered the result of the firstOr method:
```
$model = Flight::where('legs', '>', 3)->firstOr(function () {
    // ...
});
```
## Not Found Exceptions
>Sometimes you may wish to throw an exception if a model is not found. This is particularly useful in routes or controllers. The findOrFail and firstOrFail methods will retrieve the first result of the query; however, if no result is found, an Illuminate\Database\Eloquent\ModelNotFoundException will be thrown:
```
$flight = Flight::findOrFail(1);

$flight = Flight::where('legs', '>', 3)->firstOrFail();
```
## Inserting & Updating Models
### Insert
>To insert a new record into the database, you should instantiate a new model instance and set attributes on the model. Then, call the save method on the model instance:
```
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Models\Flight;
use Illuminate\Http\Request;

class FlightController extends Controller
{
    /**
     * Store a new flight in the database.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function store(Request $request)
    {
        // Validate the request...

        $flight = new Flight;

        $flight->name = $request->name;

        $flight->save();
    }
}

```
>you may use the create method to "save" a new model using a single PHP statement. The inserted model instance will be returned to you by the create method:
```
use App\Models\Flight;

$flight = Flight::create([
    'name' => 'London to Paris',
]);
```

## Updates
>To update a model, you should retrieve it and set any attributes you wish to update. Then, you should call the model's save method. Again, the updated_at timestamp will automatically be updated, so there is no need to manually set its value:
```
use App\Models\Flight;

$flight = Flight::find(1);

$flight->name = 'Paris to London';

$flight->save();
```
## Mass Updates
>Updates can also be performed against models that match a given query. In this example, all flights that are active and have a destination of San Diego will be marked as delayed:
```
Flight::where('active', 1)
      ->where('destination', 'San Diego')
      ->update(['delayed' => 1]);
```

## Deleting Models

>To delete a model, you may call the delete method on the model instance:

```
use App\Models\Flight;

$flight = Flight::find(1);

$flight->delete();
```
### Deleting An Existing Model By Its Primary Key
>if you know the primary key of the model, you may delete the model without explicitly retrieving it by calling the destroy method. In addition to accepting the single primary key, the destroy method will accept multiple primary keys, an array of primary keys, or a collection of primary keys:

```
Flight::destroy(1);

Flight::destroy(1, 2, 3);

Flight::destroy([1, 2, 3]);

Flight::destroy(collect([1, 2, 3]));

```
## Deleting Models Using Queries
>you may build an Eloquent query to delete all models matching your query's criteria.
```
$deletedRows = Flight::where('active', 0)->delete();

```

## Soft Deleting
> When models are soft deleted, they are not actually removed from your database. Instead, a deleted_at attribute is set on the model indicating the date and time at which the model was "deleted". To enable soft deletes for a model, add the Illuminate\Database\Eloquent\SoftDeletes trait to the model:
```
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class Flight extends Model
{
    use SoftDeletes;
}
```
## Restoring Soft Deleted Models
>To restore a soft deleted model, you may call the restore method on a model instance. The restore method will set the model's deleted_at column to null:
```
$flight->restore();
```
## Permanently Deleting Models
>use the forceDelete method to permanently remove a soft deleted model from the database table:
```
$flight->forceDelete();
```
# For More Detail Visit [Laravel Documentation](https://laravel.com/docs/8.x/eloquent#restoring-soft-deleted-models).
