# Unleashed API early access

We're trying to make our API compatible with the OpenAI API. This is pretty standard in this space,
so should be fairly easy to consume.

In this early release, we only support streaming output (i.e. you need to set `"stream": true`
in the request body). This is probably what you mostly want anyway, otherwise you'd have to
wait for the bot to finish generating before you see any text.

See also: https://platform.openai.com/docs/api-reference/streaming

For now, we have implemented `/models` and `/chat/completions` endpoints:

- `/models` to get a list of models available (ref: https://platform.openai.com/docs/api-reference/models)
- `/chat/completions` to generate text (ref: https://platform.openai.com/docs/api-reference/chat)

Easiest is probably to start with one of OpenAI's official client libraries:
https://platform.openai.com/docs/libraries.

## Nostr mode

You can set `"nostr_mode": true` in the request body to enable querying Nostr content.

**Limitation:** For now, the API doesn't return references to the notes (work in progress).

## Python example

The below example uses their Python library to connect to Unleashed. Node/TypeScript would be similar.

To test it out:

- Set `OPENAI_API_KEY` environment variable to your API key.
- `pip install openai`
- Run the script.

```python
from openai import OpenAI

client = OpenAI(
    base_url='https://unleashed.chat/api/v1',
)

print('Models:\n')

for model in client.models.list():
    print('- ' + model.id)

normal_stream = client.chat.completions.create(
    messages=[
        {
            'role': 'user',
            'content': 'Hello, test?',
        }
    ],
    model='dolphin-2.2.1-mistral-7b',
    stream=True,
)

print('\nNormal chat response: ', end='')

for chunk in normal_stream:
    print(chunk.choices[0].delta.content, end='')


print('\n\nNostr chat response: ', end='')

nostr_stream = client.chat.completions.create(
    messages=[
        {
            'role': 'user',
            'content': 'What has pablof7z talked about recently?',
        }
    ],
    model='dolphin-2.2.1-mistral-7b',
    stream=True,
    extra_body={
        'nostr_mode': True,
    },
)


for chunk in nostr_stream:
    print(chunk.choices[0].delta.content, end='')

print()
```
