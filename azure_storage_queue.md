
```csharp
using Azure;
using Azure.Identity;
using Azure.Storage.Queues;
using Azure.Storage.Queues.Models;
using System.Text.Json;

Console.WriteLine("Azure Queue Storage client library - .NET quickstart sample");

string queueName = "dev";
string storageAccountName = "myfilestorage";

QueueClient queueClient = new QueueClient(
    new Uri($"https://{storageAccountName}.queue.core.windows.net/{queueName}"),
    new DefaultAzureCredential());

try
{
    queueClient.SendMessage(JsonSerializer.Serialize(new MyMessage(123L, "aedfwekj")));
    Response<SendReceipt> receipt = queueClient.SendMessage("Third message");

    Console.WriteLine("\nUpdating the third message in the queue...");

    queueClient.UpdateMessage(receipt.Value.MessageId, receipt.Value.PopReceipt, Console.ReadLine());

    Console.WriteLine("\nPeek at the messages in the queue...");
    PeekedMessage[] peekedMessages = queueClient.PeekMessages(maxMessages: 10);
    var received = queueClient.ReceiveMessages(maxMessages: 10);
    foreach (PeekedMessage peekedMessage in peekedMessages)
    {
        try
        {
            MyMessage myMessage = JsonSerializer.Deserialize<MyMessage>(peekedMessage.MessageText);

            Console.WriteLine($"Message ID: {myMessage.Id}, Message: {myMessage.Message}");
        }
        catch (JsonException)
        {
            Console.WriteLine($"Message: {peekedMessage.MessageText}");
        }
    }
}
catch (RequestFailedException ex)
{
    Console.WriteLine($"Error: {ex.Message}");
}

Console.ReadLine();


record MyMessage(long Id, string Message);
```
