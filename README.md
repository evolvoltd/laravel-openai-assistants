# laravel-openai-assistants

## Get Started

First, install by using Composer
```
composer require evolvoltd/laravel-openai-assistants
```
Then, make sure to set your .env variables
```
OPENAI_API_KEY = 
GPT_ASSISTANT_ID = 
```
Note that this package uses queues. Queues need to be configured. The queue being used is 'default' by default. You can set the .env variable below to change this.
```
QUEUE_NAME = 
```
You can also set a maximum retry count. The default is 60. You can set the .env variable below to change this
```
MAX_RETRIES = 
```
## Usage
The key part of this package is the  **ask** function
```
(new GPTService)->ask($message, $parser)
```
This function will add a message to an assistant's thread, run the thread and dispatch a job to check on the run's status and retrieve the response when the run is completed.

The first parameter **'message'** requires you to pass in the message you want to send to the assistant.

The second parameter **'parser'** requires you to create and pass in a class that implements interface **ParseGPTResponse**

## ParseGPTResponse Interface
This interface has three functions:

**`public function parseResponse();`**

This function allows you to set up the functionality to parse the response from the assistant according to your requirements. Example below:
```
  public function parseResponse(string $response)
    {
        return strtoupper($response);
    }
```
**`public function executeFunctions());`**

This function allows you to set up your assistant's predefined functions for when it requests data from them. An example is displayed below. Note that the **$name** is entirely based on the functions you assigned to your assistant. The output is hardcoded in this example but could return any data of your choice.
```
  public function executeFunctions(string $name, $arguments, $toolCall)
    {
        if ($name == 'get_weather') {
            return [
                'tool_call_id' => $toolCall->id,
                'output' => "25 degrees celsius"
            ];
        } else return [];
    }
```

**`public function statusUpdate();`**

This function allows you to set up your own way to store the status and data of the run for future reference. An example is displayed below. Note that in this example, GptQuery is a model of a database table. You will need to create your own database table and model to use this function.
```
  public function statusUpdate(string $status, string $run_id, string $thread_id, string $message = null, string $file_path = null, string $file_name = null, string $response = null)
    {
        if ($status === 'new') {
            $query = GptQuery::query()->create([
                'user_id' => auth::id(),
                'status' => $status,
                'run_id' => $run_id,
                'thread_id' => $thread_id,
                'message' => $message,
                'response' => $response,
                'file_path' => $file_path,
                'file_name' => $file_name
            ]);
        } else {
            $query = GptQuery::query()->where('run_id', $run_id)->where('thread_id', $thread_id)->first();
            $query->update([
                'status' => $status,
                'response' => $response
            ]);
        }

        return $query;
    }
```

## Multiple Queues/Assistants
You can also pass in optional variables when using the **ask** function to specify the queue and assistant for the specific message sent. This allows you to use multiple queues and/or assistants in your application. An example is below:
```
(new GPTService)->ask($message, $parser, null, $assistantId, $queueName)
```
Here, **$assistantId** allows you to pass in the assistant you want to use in this request, and **$queueName** allows you to pass in the name of the queue you want to use in this request. The  **ask** function checks if these variables are provided. It uses the provided variable if available, otherwise it uses the value from the config.