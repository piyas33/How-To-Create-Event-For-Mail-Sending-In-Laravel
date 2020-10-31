# How-To-Create-Event-For-Mail-Sending-In-Laravel
How To Create Event For Mail Sending In Laravel

### Create Event
```
php artisan make:event SendMail
```

### got to app/Events/SendMail.php

```
namespace App\Events;
use App\Events\Event;
use Illuminate\Queue\SerializesModels;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
class SendMail extends Event
{
    use SerializesModels;
    public $userId;
    public function __construct($userId)
    {
        $this->userId = $userId;
    }
    public function broadcastOn()
    {
        return [];
    }
}
```
### create event listener for "SendMail" event

```
php artisan make:listener SendMailFired --event="SendMail"
```
N.B.: In this event listener we have to handle event code.

### go to app/Listeners/SendMailFired.php
```
namespace App\Listeners;
use App\Events\SendMail;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Contracts\Queue\ShouldQueue;
use App\User;
use Mail;
class SendMailFired
{
    public function __construct()
    {
        
    }
    public function handle(SendMail $event)
    {
        $user = User::find($event->userId)->toArray();
        Mail::send('emails.mailEvent', $user, function($message) use ($user) {
            $message->to($user['email']);
            $message->subject('Event Testing');
        });
    }
}
```

### Registering Events( go to app/Providers/EventServiceProvider.php)
```
namespace App\Providers;
use Illuminate\Contracts\Events\Dispatcher as DispatcherContract;
use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;
class EventServiceProvider extends ServiceProvider
{
    protected $listen = [
        'App\Events\SomeEvent' => [
            'App\Listeners\EventListener',
        ],
        'App\Events\SendMail' => [
            'App\Listeners\SendMailFired',
        ],
    ];
    public function boot(DispatcherContract $events)
    {
        parent::boot($events);
    }
}
```
### Now We are ready to use event in our controller file. (Used Event from hear)

go to app/Http/Controllers/HomeController.php
```
namespace App\Http\Controllers;
use App\Http\Requests;
use Illuminate\Http\Request;
use Event;
use App\Events\SendMail;
class HomeController extends Controller
{
    public function __construct()
    {
        $this->middleware('auth');
    }
    public function index()
    {
        $user_id = 1;
        event(new SendMail($user_id));
        return view('home');
    }
}
```
### Summary
1. Fire Event
``
event(new SendMail($user_id))
```
2. Create Event "SendMail" 
*** take $user_id and send to listener
```
    public function __construct($userId)
    {
        $this->userId = $userId;
    }
 ```

3. Create event listener for "SendMail" event
(Hear we put all code)
```
    public function handle(SendMail $event)
    {
        //code
    }
 ```
 4.Regiter Event
 ```
     protected $listen = [
        ...
        'App\Events\SendMail' => [
            'App\Listeners\SendMailFired',
        ],
    ];
```
