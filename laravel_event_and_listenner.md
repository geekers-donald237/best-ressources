# Laravel Event and Listener Example

This guide provides a step-by-step roadmap for creating an event and a listener in Laravel to send emails.

## Step 1: Create an Event

1. Generate the event file using the following command:

    ```
    php artisan make:event SendMailEvent
    ```

2. Open the `app/Events/SendMailEvent.php` file and define the event details:

    ```php
    namespace App\Events;

    use Illuminate\Queue\SerializesModels;

    class SendMailEvent
    {
        use SerializesModels;

        public $userData;
        public $status;

        public function __construct($userData, $status)
        {
            $this->userData = $userData;
            $this->status = $status;
        }
    }
    ```

## Step 2: Create a Listener

1. Generate the listener file using the following command:

    ```bash
    php artisan make:listener SendMailListener
    ```

2. Open the `app/Listeners/SendMailListener.php` file and define the listener logic:

    ```php
    namespace App\Listeners;

    use App\Events\SendMailEvent;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Support\Facades\Mail;
    use App\Mail\SendMail;

    class SendMailListener implements ShouldQueue
    {
        use InteractsWithQueue;

        public function handle(SendMailEvent $event)
        {
            $userData = $event->userData;
            $status = $event->status;

            // Choose email content based on the status
            $view = 'emails.sendmail.default';

            if ($status == 'status1') {
                $view = 'emails.sendmail.status1';
            } elseif ($status == 'status2') {
                $view = 'emails.sendmail.status2';
            } elseif ($status == 'status3') {
                $view = 'emails.sendmail.status3';
            } // or other logic for sending mail view 

            // Send the email
            Mail::to($userData['email'])->send(new SendMail($userData, $status, $view));
        }
    }
    ```

## Step 3: Create a Mailable

1. Generate the mailable file using the following command:

    ```bash
    php artisan make:mail SendMail
    ```

2. Open the `app/Mail/SendMail.php` file and define the mailable logic:

    ```php
    namespace App\Mail;

    use Illuminate\Bus\Queueable;
    use Illuminate\Mail\Mailable;
    use Illuminate\Queue\SerializesModels;

    class SendMail extends Mailable
    {
        use Queueable, SerializesModels;

        public $userData;
        public $status;
        public $view;

        public function __construct($userData, $status, $view)
        {
            $this->userData = $userData;
            $this->status = $status;
            $this->view = $view;
        }

        public function build()
        {
            return $this->view($this->view)
                        ->with(['userData' => $this->userData]);
        }
    }
    ```

## Step 4: Configure Event-Listener Mapping

1. Open the `App/Providers/EventServiceProvider.php` file.

2. Add the event-listener mapping in the `$listen` property:

    ```php
    protected $listen = [
        SendMailEvent::class => [
            SendMailListener::class,
        ]
    ];
    ```

## Step 5: Trigger the Event

Create a route or a controller method where you want to trigger the event:

```php
Route::get('/trigger-email', function () {
    $userData = [
        'name' => 'John Doe',
        'email' => 'john.doe@example.com',
        // ... other necessary data
    ];

    event(new \App\Events\SendMailEvent($userData, 'status1'));

    return 'E-mail sent successfully!';
});
```

Replace the data in `$userData` with the appropriate values for your use case.



