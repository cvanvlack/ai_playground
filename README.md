# ai_playground

Because I didn't want to create new venv and .env for every tutorial, I am running everything from the top level folder.

## Quickstart
[Quick Start Link](https://platform.openai.com/docs/quickstart?context=python)

Setup (MacOS)
```
python -m venv openai-env
source openai-env/bin/activate
```
Note that we are using .env and the dotenv package
```
pip install python-dotenv
```
To run the examples
```
python quickstart/quickstart_chat_completions.py 
python quickstart/quickstart_embeddings.py
python quickstart/quickstart_images.py 
```

## Tutorials

###[Creating an automated meeting minutes generator with Whisper and GPT-4](https://platform.openai.com/docs/tutorials/meeting-minutes)

The first time I tried this, I used (VLC to extract the audio from a recorded meeting)[https://researchguides.case.edu/c.php?g=1286426]. However, the file was 53.4 MB and was almost 56 minutes long.

[This forum thread](https://community.openai.com/t/questions-regarding-transcribing-long-audios-25mb-in-whisper-api/267384) mentioned that chopping up the audio into chunks needs to be done a little cleverly so you don't chop the audio right in the middle of a word. The pointed to a [Whisper with Jax](https://github.com/sanchit-gandhi/whisper-jax) repo.

Note that we also dumped our outputs at various stages in case a later stage failed as we didn't want to double pay for any services. 

Just to keep things moving, [I used this guide with Audacity](https://recorder.easeus.com/screen-recording-resource/how-to-cut-audio-in-audacity.html) to figure out how to cut the audiio into chunks (I tried VLC but couldn't get it to work). Audacity showed the relevant breakpoints.

I kept getting the error 
```
return transcription['text']
TypeError: 'Transcription' object is not subscriptable
```
The transcription also had a problem with one of my colleagues accents and started using a language that was completely incorrect. I tried changing the call to
```
transcription = client.audio.transcriptions.create(
            model="whisper-1",
            file= audio_file,
            response_format="json"
            language="en"
            )
        print(transcription)
```
This still prevented me from looping over the transcription objects.

I ended up finding a workaround for the meeting minutes by reducing the audio quality. I opened Audacity and then used `File > Export Audio` and I set `Sample Rate = 8000` and `Quality = Medium, 145-185 kbps`. This got the .mp3 file down to 10.9MB.

However, after trying that I hit some [rate limits](https://platform.openai.com/account/rate-limits).
```
openai.RateLimitError: Error code: 429 - {'error': {'message': 'Rate limit reached for gpt-4 in organization on tokens per min (TPM): Limit 10000, Used 3172, Requested 9505. Please try again in 16.062s. Visit https://platform.openai.com/account/rate-limits to learn more.', 'type': 'tokens', 'param': None, 'code': 'rate_limit_exceeded'}}
```
I waited a few minutes and then only dealt with a single send to the API (to take the transript and extract the abstract and I then got)
```
openai.BadRequestError: Error code: 400 - {'error': {'message': "This model's maximum context length is 8192 tokens. However, your messages resulted in 8632 tokens. Please reduce the length of the messages.", 'type': 'invalid_request_error', 'param': 'messages', 'code': 'context_length_exceeded'}}
```
From the [models documentation](https://platform.openai.com/docs/models/gpt-4-and-gpt-4-turbo) the gpt-4 model has a limit of `8,192 tokens`. As such, I switched to `gpt-4-32k`. 

I then got the error
```
openai.NotFoundError: Error code: 404 - {'error': {'message': 'The model `gpt-4-32k` does not exist or you do not have access to it. Learn more: https://help.openai.com/en/articles/7102672-how-can-i-access-gpt-4.', 'type': 'invalid_request_error', 'param': None, 'code': 'model_not_found'}}
```
so I tried ` gpt-3.5-turbo-1106` which finally worked!