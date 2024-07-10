# ADR: Detect Potential harmful text

**Context**: Our objective is to implement a solution for identifying potentially harmful text within the Feedback Circuit. The solution must meet the following criteria:

1. Detect harmful content entered by user, whether public or anonymous.
2. Provide warnings or suggestions to users based on detected harmful content.

**Problem**:

1. Identifying harmful text input by users.
2. Categorizing harmful text and providing appropriate suggestions to users.

**Considered Options**:

1. **[Open AI:](https://platform.openai.com/docs/guides/moderation)**: The moderations endpoint is a tool you can use to check whether text is potentially harmful. Developers can use it to identify content that are might be harmful and take action, for instance by filtering it.
   The models classifies the following categories: 1. hate, 2. hate/threatening, 3. harassment, 4. harassment/threatening, 5. self-harm, 6. self-harm/intent, 7. self-harm/instructions, 8. sexual, 9. sexual/minors, 10. violence, 11. violence/graphic.
   The model responds with the flagged property which is a boolean and it sets to true if the text sentiment lies in any of the category along with the all the categories and their respective scores.
   The moderation endpoint is free to use for most developers. For higher accuracy, try splitting long pieces of text into smaller chunks each less than 2,000 characters.
2. **[Clodflare AI:](https://developers.cloudflare.com/api/operations/workers-ai-post-run-model)**: The Workers AI model i.e `distilbert-sst-2-int8` which is used to detect potentailly harmful text for sentiment classification. The model responds with two properties i.e positive and negative along with their scores and the property names are self explanatory.
3. **[Google Gemini:](https://ai.google.dev/api/rest/v1/GenerateContentResponse)**: The Google Gemini AI for text moderation.
4. **[NPM packages:](https://www.npmjs.com/package/bad-words)**: Multiple node packages for text moderation.

**Discarded Options:**

1. **Clodflare AI:**

   1. The model `distilbert-sst-2-int8` used for detection of text moderation responds with only two properties which might not be sufficient enough to provide suggestion.
   2. The Cloudflare will start charging $0.011/1,000 requests from April 1, 2024, after all usage exceeding 10,000 requests for text classification.

2. **Google Gemini:**

   1. The Gemini AI doesn't have any text moderation endpoint to detect any potential harmful text.

3. **NPM Packages:**

   1. The node packages supports mostly only for bad words and not any other sentimental text.

**Decision**: We'll use the **Open AI** for the following reasons -

1. The Text moderation Endpoint is pretty easy to use and it is well documented.
2. The response contains various categories along with scores which can be used for suggestion and warnings.
3. Open AI does not impose any known limitations on the usage of their moderation API.

**Consequences**:

1. Team members will need to familiarize themselves with the Open AI's text moderation endpoints.

**References**:
N/A
