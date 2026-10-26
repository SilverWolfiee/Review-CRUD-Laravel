# WebDev 101 w/ Laravel  
Make sure smua ini ud ad:  
```sh
php -v
composer -V
node -v
npm -v
```
## Setup:  
1. Run `composer create-project laravel/laravel <project-name>` 
    buat bikin project laravel ny
    * Note: Laravel punya BINUS itu v11 tpi klo dicoba create-project pake v11 g bs/g dikasih kykny grgr 2 vulnerability CVE 9 yg kmrn :p  
    Tpi shrsny pake v13 g mslh; semua function ny msh bsbs aj.
    


2. Buat contoh ini w pake project-name = laravel-CRUD. cd ke project dir ny (`cd laravel-CRUD`)

3. Run `composer require laravel/ui && php artisan ui bootstrap` buat install & setup scaffolding bootstrap. Abis run command ini, di folder `resources` bakal muncul folder `sass` trus di folder `js` bakal muncul file `bootstrap.js`

4. Commonly: configuration static files (alias file css & js) utk sebuah project itu ad di 1 file yg namany `app.<css|scss|js>` utk styles & scripts. Di `resources/js/` kn ad file `bootstrap.js`. Import file itu ke `app.js` dgn cara add line: `import './bootstrap'`
    * Note: `import './bootstrap';` dan `import 'bootstrap';` itu beda.  
    `'bootstrap'` itu package bootstrap yg ad dri npm node_modules. `'./bootstrap'` itu file bootstrap.js yg ad di directory yg sama. Notice that `resources/js/bootstrap.js` ud import bootstrap duluan jd lgsg import ./bootstrap aj

5. `php artisan ui bootstrap` ud download config buat bootstrap ny tpi packages ny sendiri blom di download. Run `npm install && npm install axios`(file bootstrap.js perlu axios) buat download packages ny.

6. Run `php artisan serve` buat make sure the app works. Welcome page ny may or may not look a lil fucked up but that's fine we're not gonna use that shi anyway yg penting g ad error dlu. Kebetulan Laravel skrg itu an "agent first" web framework whatever the hell that means basically laravel skrg di optimize utk vibe coding with maximum efficiency jd klo ad error di kanan atas ad 'Copy as Markdown'. Copy itu aj trus kasih ke codex jgn tny w w hantem lu


Alternatively, do `git clone https://git.tensorcode.co.id/SilverWolf/template-laravel.git`
buat dapat template laravel yang di kasih binus.
then just do : 
`npm install && npm install axios`
`php artisan serve`

## Setup Xampp
1. On windows, just open xampp app, and point it to your .env
2. on linux, do `sudo /opt/lampp/manager-linux-x64.run`
3. Create new database in phpmyadmin with the same name as the one in .env

> If Laravel shows `MissingAppKeyException` or “No application encryption key has been specified,” generate an application key with `php artisan key:generate` from the project directory.

## Basic CRUD Web App | Tasks Manager
1. First thing to do when making a CRUD (noun) app is making a DB Model for the app to CRUD (verb).  
Run: `php artisan make:model Task --migration --factory --seed`  

| Command/Flag | Shit |
| --- | --- |
| `php artisan make:model` | Use artisan to make the DB Model |
| Task | nama Model ny |
| `--migration` | Automatically make the migration for the model |
| `--factory` | Automatically make the factory for the model |
| `--seed` | Automatically make the seeder for the model |

2. Next is to populate the Model and it's related table. Buat Task kt, kt pake 3 attribute aj:
    * title -> string
    * description -> string
    * completed -> boolean

Di file Model ny (`app/Models/Task.php`):
```php
class Task extends Model
{
    /** @use HasFactory<\Database\Factories\TaskFactory> */
    use HasFactory;

    // Tambahin fillable ama casts ini
    protected $fillable = [
        'title',
        'description',
        'completed'
    ]; 
    /**
     * $fillable itu eloquent (ORM ny laravel) convention yg ngedefine attribute dri model yg bs di assign secara manual oleh user (contoh: pake form method post html). 
     * Knp perlu ini karena ad bbrp case dmn sebaikny kt jangan ksh user manually define attribute sebuah model 
     * (contoh: klo misal kt bikin User sbg FK dri Task (User--> has many of--> Task) kt g mau user bs manually define user mana yg bikin sebuah task. 
     * Seharusnya otomatis user yg buat task ny adl user yg jd FK task ny.)
     * Kt g ush setting data type ny di sini soalny itu di atur ama table DB & controller ny 
     */

    protected $casts = [
        'completed' => 'boolean'
    ];
    /**
     * Ini w rasa g penting tpi good to know aj. $casts itu eloquent convention jg dmn kt pastiin sebuah attribute direturn dlm php compatible format 
     * (contoh: tergantung pake DB apa sebuah boolean bs di return sbg true/false True/False 1/0. 
     * $casts ini make sure klo attribute completed itu direturn dlm format boolean ny php alias true/false)
     */

    /**
     * Notice dua dua ny itu protected krn kt g mau something yg g related or worse: malicious bs access and/or change value dr attribute tsb. 
     * Why not private? Class ini msh harus bs diaccess ama Eloquent buat dia ngatur relationship antara PHP ama DB ny. 
     * Klo protected msh bs diaccess ama parent/child classes ny. Grgr Task extends Model dmn Model itu sebuah Class Eloquent, Eloquent bs pake. 
     * Klo private cm bs diaccess di dlm class sndiri jd Eloquent g bs pegang but if you wanna deal with the complex SQL to manage the data yourself go ahead --_--
     */
}
```

Di file migration ny (`database/migrations/<some-date-and-time>_create_tasks_table.php`):
```php
return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('tasks', function (Blueprint $table) {
            $table->id();
            $table->timestamps();
            // Add these: $table-><data type>('<name>')-><any extra argument>
            $table->string('title');
            $table->string('description')->nullable(); // nullable alias user g perlu taro description. Klo title harus
            $table->boolean('completed')->default(false);
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('tasks');
    } // ignore this. Ini basically function yg bakal di execute klo kt mau rollback migrate ny.
};
```

Klo uda: `php artisan migrate`

3. Klo entity DB ny ud kelar, selanjutny kt bikin controller ny. Good rule of thumb itu basically buat 1 controller buat setiap DB model yg lu bikin. Kt cuma ad 1 jd kt bikin 1 aj: `php artisan make:controller TaskController --model=Task --resource`

| Command/Flag | Shit |
| --- | --- |
| `php artisan make:controller` | Use artisan to make the Controller |
| TaskController | nama Controller ny. Naming convention laravel itu (nama class DB Model ny)+Controller |
| `--model=Task` | Explicitly state the related Model for this Controller |
| `--resource` | Automatically create boilerplates for basic CRUD functions (index, create, store, etc. klo misal flag ini di laravel binus g bisa ya gg lah ketik sndiri aj g susah kok) |

The Controller (ideally) should control everything concerning the Model it's related to, including (but not limited to) rendering views and backend operations/logic. Which requests go to which method is controlled by the router (`routes/web.php`). We'll take care of the controller methods (`app/Http/Controllers/TaskController.php`) first:

> Fair warning: Convention convention laravel itu agk autistic ygy jd good practice laravel itu lu pny 2 method per operation yg ad logic ny: 1 buat render view ny (contoh: create, edit) and 1 more buat actual operation ny (contoh: store, update)

```php
class TaskController extends Controller
{
    /**
     * Display a listing of the resource.
     */
    public function index()
    {
        $tasks = Task::latest()->get();
        // Klo mo ambil bbrp doank:
        // $tasks = Task::latest()->take(<bbrp>)->get();

        return view("index", compact("tasks")); // compact itu buat pass sebuah variable ke dlm view/template ny.
        // Compact itu infer variable yg di pass berdasarkan nama jd hrs bener. 
        // Klo variable ny $ihatelaravel compact ny itu nama variable ny tp g ad '$' ny (compact('ihatelaravel')) 
    }

    /**
     * Show the form for creating a new resource.
     */
    public function create()
    {
        return view("create");
    }

    /**
     * Store a newly created resource in storage.
     */
    public function store(Request $request)
    {
        $validated = $request->validate([
            'title' => ['required', 'string'],
            'description' => ['nullable', 'string'],
            'completed' => ['boolean']
        ]);
        /**
         * Td kn w ad bilang kt g set data types ny di class Model ny grgr itu bakal diatur ama DB table and controller ny, y jd ini tu itu y. 
         * Kt control pake function validate(). Notice function validate itu ngambil 1 argument yaitu array. Isi array ny itu kyk dictionary dmn 
         * key ny itu nama attribute/variable ('title', 'description', 'completed') ny trus value ny itu array yg isiny argument yg kt bakal pake buat validate request ny ('required', 'string', 'nullable', etc.). 
         * Klo lu tanya knp yg ini pake => and not -> ya basically -> itu buat access isi dri sebuah variable jd misal task kn ad title jd klo mo access title ny itu $task->title 
         * klo => itu lowk i have no clue basically yg g pake -> itu pake => ud gt aj
         */

        Task::create($validated);

        return redirect()->route('index');
    }

    // /**
    //  * Display the specified resource.
    //  */
    // public function show(Task $task)
    // {
    //     // Ini tu method buat show individual resource alias 1 page buat showcase 1 task doank. 
    //     // Sbnrny g penting buat project kt tp klo lu penasaran kode ny kyk gini doank
    //     return view('show', compact('task'));
    // }

    /**
     * Show the form for editing the specified resource.
     */
    public function edit(Task $task)
    {
        return view('edit', compact('task'));
    }

    /**
     * Update the specified resource in storage.
     */
    public function update(Request $request, Task $task)
    {
        $validated = $request->validate([
            'title' => ['required', 'string'],
            'description' => ['nullable', 'string'],
            'completed' => ['boolean']
        ]);

        $task->update($validated);

        return redirect()->route('index');
    }

    /**
     * Remove the specified resource from storage.
     */
    public function delete(Task $task) // ini awalny destroy tp w ganti jd delete biar lbh consistent
    {
        $task->delete();

        return redirect()->route('index');
    }

    public function markAsComplete(Task $task) // ini function custom kt bikin sndiri
    {
        $task->update([
            'completed' => true
        ]);

        return redirect()->route('index');
    }
}
```

Skrg routing (`routes/web.php`)

```php
use App\Http\Controllers\TaskController; // jgn lupa import controller yg bakal dipake

Route::get('/', function () {
    return view('welcome');
}); // Route yg ini ama `resources/views/welcome.blade.php` delete aj kt g pake

Route::get('/', [TaskController::class, 'index'])
    ->name('index');

Route::get('/create', [TaskController::class, 'create'])
    ->name('create');

Route::post('/create/store', [TaskController::class, 'store'])
    ->name('store');

// Route::get('/show/{task}', [TaskController::class,'show'])
// ->name('show');
// ini buat function show td yg g kepake tp klo lu penasaran, code ny begini doank

Route::get('/edit/{task}', [TaskController::class, 'edit'])
    ->name('edit');

Route::put('/edit/update/{task}', [TaskController::class, 'update'])
    ->name('update');

Route::delete('/delete/{task}', [TaskController::class, 'delete'])
    ->name('delete');

Route::patch('edit/markAsComplete/{task}', [TaskController::class, 'markAsComplete'])
    ->name('markAsComplete');
```

| HTTP Method | Purpose | Task example |
|---|---|---|
| GET | Read data or display a page without changing data. | List tasks or display a form. |
| POST | Submit data, commonly to create a new resource. | Save a new task. |
| PUT | Conventionally replace an entire resource. | Submit a task's complete set of editable fields. |
| PATCH | Update part of a resource. | Change only a task's completion status. |
| DELETE | Remove a resource. | Delete a task. |

> Laravel does not automatically enforce full replacement for PUT or partial updates for PATCH; your controller determines how submitted fields are handled.  
> HTML forms support only GET and POST. For PUT, PATCH, or DELETE, use method="POST" with Blade's @method('PUT'), @method('PATCH'), or @method('DELETE'). Include @csrf in all forms that change data.

## Backend uda slsai
Skrg mulai bikin view ny/web page ny. Kt kn pake bootstrap tp w g tau pasti botstrap lu org version brp jd check pake `npm list bootstrap` trus buka documentation bootstrap buat version yg bener. Versi bootstrap w itu `bootstrap@5.3.8` jd w pake v5.3  
> I ain't writing the HTML in here so be a good boy/girl and look inside `resources/views` yourself


## Factories and Seeders (Faker)

A **factory** describes how to create fake model attributes. Laravel uses **Faker** to generate realistic sample values. A **seeder** runs factories (or other database setup code) to insert records into the database.

### Create them for an existing model

The `--factory` and `--seed` flags on `make:model` are convenient when creating a new model. If the model already exists, generate the files separately. For the `Task` model:

```sh
php artisan make:factory TaskFactory --model=Task
php artisan make:seeder TaskSeeder
```

Laravel includes Faker in a standard project as a development dependency, so it is usually ready to use. If your project does not have it, install it with `composer require fakerphp/faker --dev`.

### Define fake task data

Edit `database/factories/TaskFactory.php`. Return an array whose keys match the columns in the `tasks` table:

```php
public function definition(): array
{
    return [
        'title' => fake()->sentence(),
        'description' => fake()->paragraph(),
        'completed' => fake()->boolean(),
    ];
}
```

The model must call `HasFactory` trait for `Task::factory()` to work. The `Task` model created with Laravel's factory flag normally already has it:

```php
use Illuminate\Database\Eloquent\Factories\HasFactory;

class Task extends Model
{
    use HasFactory;
}
```

### Use the factory in a seeder

Edit `database/seeders/TaskSeeder.php` and import the model with its `App\Models` namespace:

```php
use App\Models\Task;
use Illuminate\Database\Seeder;

class TaskSeeder extends Seeder
{
    public function run(): void
    {
        Task::factory()->count(10)->create();
    }
}
```

This creates and saves 10 tasks. Check that the factory attributes match your migration and that any required fields have values.

### Run the seeder

To run only the task seeder:

```sh
php artisan db:seed --class=TaskSeeder
```

To run it along with the application's other seeders, add it to `database/seeders/DatabaseSeeder.php`:

```php
public function run(): void
{
    $this->call(TaskSeeder::class);
}
```

Then run:

```sh
php artisan db:seed
```

`db:seed` runs `DatabaseSeeder`. If that file creates a fixed user such as `test@example.com`, running it again can fail because the email is already in the database. Either run `TaskSeeder` directly or make that user creation repeatable with `User::firstOrCreate(...)`.

For a development database, you can recreate all tables and seed them in one command:

```sh
php artisan migrate:fresh --seed
```

**Warning:** `migrate:fresh` drops all tables and their data before recreating them. Do not use it on a database whose data you need to keep.
