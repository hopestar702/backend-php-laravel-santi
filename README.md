
| Laravel Vuetable  |  L5.4            | L5.5             | L5.6             | L5.7             | L5.8             | L6               | L7               | L8               |
|-------------------|------------------|------------------|------------------|------------------|------------------|------------------|------------------|------------------|
| < 1.0             |:heavy_check_mark:|:heavy_check_mark:|:heavy_check_mark:|:heavy_check_mark:|:heavy_check_mark:|:x:               |:x:               |:x:               |
| \> 1.0            |:x:               |:x:               |:x:               |:heavy_check_mark:|:heavy_check_mark:|:heavy_check_mark:|:heavy_check_mark:|:heavy_check_mark:|

## Installation
1. Run the composer require command from your terminal:

        composer require "santigarcor/laravel-vuetable"

2. If you laravel version not supported the package discovery, set in your `config/app.php`:
    - In the providers array add:

            Vuetable\VuetableServiceProvider::class,

    - In the aliases array add:

            'Vuetable' => Vuetable\VuetableFacade::class,

## Usage
Your request to the controller should have this data:

```javascript
{
    sort: '', // column_name|asc or column_name|desc
    page: 1,
    per_page: 10,
    searchable: [
        // This array should have the names of the columns in the database
    ],
    filter: '' //The text that is going to be used to filter the data
}
```

You can also specify the sorting order using the "order" attribute (required by https://mannyyang.github.io/vuetable-3/ ):
```javascript
{
    sort: '', // column_name
    order: '', // asc or desc
}
```


So for example lets create the table for the users with their companies. Then in the javascript we should have:

```javascript
data = {
    sort: 'users.name|asc',
    page: 1,
    per_page: 10,
    searchable: [ // This means the 'users.name', 'users.email' and 'companies.name' columns can be filtered through the 'filter' attribute in the data.
        'users.name',
        'users.email',
        'companies.name',
    ]
}

axios.get('http://url.com/users-with-companies', data)
```

In Controller we can provide Eloquent:

```php
class UsersDataController extends Controller
{
    public function index() {

        $query = User::select([
                'users.id',
                'users.name',
                'users.email',
                'companies.name as company',
                'companies.company_id'
            ])
            ->leftJoin('companies', 'users.company_id', '=', 'companies.id');

        return Vuetable::of($query)
            ->editColumn('company', function ($user) {
                if ($user->company) {
                    return $user->company;
                }

                return '-';
            })
            ->addColumn('urls', function ($user) {
                return [
                    'edit' => route('users.edit', $user->id),
                    'delete' => route('users.destroy', $user->id),
                ];
            })
            ->make();
    }
}
```

Or Collection
```php
class UsersDataController extends Controller
{
    public function index() {

        $query = new Collection([
             ['name' => 'John Doe', 'email' => 'john@mail.com'],
             ['name' => 'Jane Doe', 'email' => 'jane@mail.com'],
             ['name' => 'Test John', 'email' => 'test@mail.com']
        ]);

        return Vuetable::of($query)
            ->editColumn('name', function ($user) {
                return Str::lower($user['name']);
            })
            ->addColumn('urls', function ($user) {
                return [
                    'edit' => route('users.edit', $user['id']),
                    'delete' => route('users.destroy', $user['id']),
                ];
            })
            ->make();
    }
}
```
This controller is going to return:
```json
{
  "current_page": 1,
  "from": 1,
  "to": 10,
  "total": 150,
  "per_page": 10,
  "last_page": 15,
  "first_page_url": "http://url.com/users-with-companies?page=1",
  "last_page_url": "http://url.com/users-with-companies?page=15",
  "next_page_url": "http://url.com/users-with-companies?page=2",
  "prev_page_url": null,
  "path": "http://url.com/users-with-companies",
  "data": [
    {
      "id": 1,
      "name": "Administrator",
      "email": "administrator@app.com",
      "company": "-",
      "company_id": null,
      "urls": {
        "edit": "http://url.com//users/1/edit",
        "delete": "http://url.com//users/1"
      },
    },
    {
      "id": 2,
      "name": "Company Administrator",
      "email": "company_administrator@app.com",
      "company": "-",
      "company_id": null,
      "urls": {
        "edit": "http://url.com//users/2/edit",
        "delete": "http://url.com//users/2"
      },
      ...
    }
  ],
}
```
