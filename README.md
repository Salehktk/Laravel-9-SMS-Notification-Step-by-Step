# Laravel SMS Notification Code Step by Step
In this post, I will show you how to implement Laravel SMS notification using Vonage API (formerly known as Nexmo). Sometimes we need to implement a notification that will directly send an SMS to your user transaction.

## System Requirements

* Windows 10
* XAMPP, WAMPP server [Download](https://www.apachefriends.org/download.html)
* PHP 7.4 or better
* nodejs [Download](https://nodejs.org/en/download/)
* composer [Download](https://getcomposer.org/Composer-Setup.exe)

Laravel is smoothly working with Vonage API so we will use it as our SMS provider. So if you don't have an account just register  [here](https://ui.idp.vonage.com/ui/auth/secondFA?context=eyJoeWRyYUZsb3ciOiI3OTYzODFkMDEyMDU0OWIzYTcwZmJjYTQ2YzRjZDkyZSIsInN1YmplY3QiOiIiLCJmb3JjZUlkUCI6IiIsIm9pZGNDb250ZXh0Ijp7ImFjcl92YWx1ZXMiOm51bGwsInVpX2xvY2FsZXMiOm51bGx9LCJjbGllbnRJRCI6ImRhc2hib2FyZC1tYWluIiwicmVnaXN0cmF0aW9uRmxvdyI6IiIsImxvZ2luRmxvdyI6InhrZ3JsRXQvdzloRjQwWkxsMW9zZHlabnR6aWtLUHJKK29VYWd0ZktGY2RKRklUeUNCZUY5MWF3NHVuRXpWY0M1U2RhL0NjcnBZK0RoaVNoYmhpNTVaN0hCZGE0YkNnYXRBbmpHVnB5RjlRd1dkQmpnWmJ3ODJWWmxnMERHVUdHIiwiYWRkcmVzc1ZlcmlmeSI6bnVsbCwiZm9yY2VMb2dnZWRPdXQiOmZhbHNlLCJmbG93VHJhY2VJZCI6IjY1NzhmZWVlLTQxMTgtNDhjNi1hOTA2LTBiZGYyNWY5MWJmNyIsInBob25lVmVyaWZpZWQiOnRydWV9)

Now let's start.

## Step 1: Laravel Installation

If you don't have a Laravel install in your local just run the following command below:
   
    composer create-project --prefer-dist laravel/laravel laravel-sms-notification

## Step 2: Database Configuration

If your **Laravel project** is fresh then you need to update your database credentials. Just open the .env file in your Laravel  project.

**.env**

         DB_CONNECTION=mysql
         DB_HOST=127.0.0.1
         DB_PORT=3306
         DB_DATABASE=your_database_name_here
         DB_USERNAME=your_database_username_here
         B_PASSWORD=your_database_password_here
    
## Step 3: Migration Setup

Here we need to modify first the User table before running the migrations

Because we are using phone_number from the user table we need to add this field from your user migration. See below example code.

        <?php

        use Illuminate\Database\Migrations\Migration;
        use Illuminate\Database\Schema\Blueprint;
        use Illuminate\Support\Facades\Schema;

        class CreateUsersTable extends Migration
        {
            /**
             * Run the migrations.
             *
             * @return void
             */
            public function up()
            {
                Schema::create('users', function (Blueprint $table) {
                    $table->id();
                    $table->string('name')->nullable();
                    $table->string('email')->unique();
                    $table->string('phone_number')->unique();
                    $table->timestamp('email_verified_at')->nullable();
                    $table->string('password');
                    $table->softDeletes();
                    $table->rememberToken();
                    $table->timestamps();
                });
            }

            /**
             * Reverse the migrations.
             *
             * @return void
             */
            public function down()
            {
                Schema::dropIfExists('users');
            }
        }
        
 If your user already exists then you must add your phone_number column using migration.
 
 Then once done. Kindly run the following command:
 
     php artisan migrate
     
 Then once done let's create a seeder for our user. Run the following command:
 
     php artisan make:seeder CreateUsersSeeder
     
 Once our seeder is generated kindly to the **database/seeders** directory. Open the CreateUsersSeeder.php and you will see the following code:
 
 
         <?php

        namespace Database\Seeders;

        use App\Models\User;
        use Illuminate\Database\Seeder;

        class CreateUsersSeeder extends Seeder
        {
            /**
             * Run the database seeds.
             *
             * @return void
             */
            public function run()
            {
                User::create([
                    'name' => 'XYZ',
                    'email' => 'email@gmail.com',
                    'phone_number' => '+923139xxxxxx',
                    'password' => bcrypt('password')
                ]);
            }
        }
        
Then run the following command:

    php artisan db:seed --class=CreateUsersSeeder
    
## Step 4: Install Package & Connect Vonage API

To work with SMS notifications using Vonage we need to install Vonage channel notification via composer:

    composer require laravel/nexmo-notification-channel
    
Once installed let's connect Vonage API to our Laravel App. Login to your Vonage account and click "Getting started".

![image](https://user-images.githubusercontent.com/74606108/186154091-b36985d8-5c30-4c97-80fe-bdcc4dd98678.png)

Then add the credentials to your **.ENV** file.

        NEXMO_KEY=key_here
        NEXMO_SECRET=secret_here
        
Next, we will add sms_from it to our config/services.php file. The sms_from is the phone number to use for sending messages and will be sent from.

      'nexmo' => [
         'sms_from' => 'Vonage SMS API HERE',
      ],
        
 ## Step 5: Setting Up Controller
 Run the bellow commond for HomeController 
 
    php artisan make:controller HomeController
    
In this section, we will add SMS notification function in HomeController. See below the complete code of our controller:

      <?php

      namespace App\Http\Controllers;

      use App\Models\User;

      class HomeController extends Controller
      {
          public function send()
          {
              $user = User::first();
              $basic  = new \Nexmo\Client\Credentials\Basic('595e3f0f', 'zihbVfSTJRzo3vPh');
              $client = new \Nexmo\Client($basic);

              $message = $client->message()->send([
                  'to' => $user->phone_number,
                  'from' => 'XYZ',
                  'text' => 'Hi, Dear Sir how are you '
              ]);

              dd('SMS has been sent successfully!.');
          }
      }
      
## Step 6: Setting Up User Model

      <?php

      namespace App\Models;

      use Illuminate\Database\Eloquent\Factories\HasFactory;
      use Illuminate\Foundation\Auth\User as Authenticatable;
      use Laravel\Sanctum\HasApiTokens;

      class User extends Authenticatable
      {
          use HasApiTokens, HasFactory;

          /**
           * The attributes that are mass assignable.
           *
           * @var string[]
           */
          protected $fillable = [
              'name',
              'email',
              'password',
              'phone_number',
          ];

          /**
           * The attributes that should be hidden for serialization.
           *
           * @var array
           */
          protected $hidden = [
              'password',
              'remember_token',
          ];

          /**
           * The attributes that should be cast.
           *
           * @var array
           */
          protected $casts = [
              'email_verified_at' => 'datetime',
          ];
      }
      
## Step 7: Setting up Routes

Next, we will create a route for our SMS notification sending. Just open the **routes/web.php** file and add the following routes.

      <?php

      use Illuminate\Support\Facades\Route;

      /*
      |--------------------------------------------------------------------------
      | Web Routes
      |--------------------------------------------------------------------------
      |
      | Here is where you can register web routes for your application. These
      | routes are loaded by the RouteServiceProvider within a group which
      | contains the "web" middleware group. Now create something great!
      |
      */

      Route::get('/', function () {
          return view('welcome');
      });


      Route::get('/send', '\App\Http\Controllers\HomeController@send')->name('home.send');
  
 You can test it now by running the serve command:
 
    php artisan serve
    
 Then run the URL below to your browser to send an SMS notifications to your user.
 
    http://127.0.0.1:8000/send
   
Now you will see this output in your phone.

![image](https://user-images.githubusercontent.com/74606108/186329267-58b4d64b-75c9-4c08-8045-34967d25f866.png)

 I hope it helps. Thank you for reading :)



     
 
