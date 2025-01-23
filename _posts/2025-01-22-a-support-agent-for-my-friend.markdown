---
layout: post
title: "A Support Agent for My Friend"
date: 2025-01-22 16:36:02 +0100
categories: jekyll update
---

I've been brainstorming ideas to build! I promised myself to focus on thinking before jumping straight to building—no rushing this time.

While waiting for inspiration, I decided to help out my slightly more successful friend (I am not crying, you are). My friend Ludwig is building [FITS](https://www.fits-app.com). Not the most exciting idea, but hey, at least he has a few hundred thousand downloads. 🎉

Ludwig’s an indie dev, and like many, he _really_ doesn’t want to deal with support tickets. He’s especially annoyed with Intercom charging $1 per ticket resolution. So greedy! 😡

![](/assets/images/ludwighenne-linkedin.png){: width="400" }

Since I’ve been mostly scrolling through Hacker News, I figured I’d put my time to good use and help him out.

### Ludwig’s First Request

- **Triage tickets:** Decide whether to answer directly or escalate.
- **Reply in the original language** of the user.
- **Forward unresolved tickets via email.**
- Add a **“send to human”** button for when the bot isn’t helpful.

### My Plan

Here’s how I’m approaching this project:

1. **Build a Q&A dataset**
   - Extract Ludwig's past support emails and create an array of common questions and answers.
2. **Set Up a RAG Pipeline**
   - Use a Retrieval-Augmented Generation (RAG) pipeline to give the chatbot context for its replies.
3. **Implement Function Calling**
   - Forward unresolved issues to Ludwig via email automatically.

## How does a RAG work?

If you have built a chatbot before, you know how hard it is to get it to answer all of the questions properly. You tune your prompt and bam it works for the new question, you try something that was working before and it does not anymore.. :(

Good thing: with a rag pipeline you can align your chatbot to the context, and it works wonderfully.

The way it works is the following

1. User asks a question
2. We look for the most similar question in our dataset
3. Instead of sending just the user question to the model, we send the user question and the most similar question and answer from our dataset
4. You are literally sam altman

Some magic happens between step 1 and 2. To compare questions and answers we use embeddings. If you don't know what embeddings are, please watch this. I wish I could erase my memory to watch this video again.

<iframe width="560" height="315" src="https://www.youtube.com/embed/wjZofJX0v4M?si=cDhwpdehmTICRb21&amp;start=749" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
<br>
## Building a dataset

We put all of cleaned up the emails into a notion page of questions and answers. One pair example:

![](/assets/images/qna.png){: width="700" }

After getting over 23 pairs, I've decided I was ready to test. I made a script to get the embeddings for each question. Since I just want this to work for Ludwig, I made some calls so we could ship this fassssst:

- My vector database is going to be a txt file.
- I will use the openai embeddings api.

```javascript
const qnaArray = [
  {
    question:
      "I paid for a subscription, now it asks me again...",
    answer:
      "Please make sure you log in with the same accoun....",
  },
  ...
];

// Function to get embeddings for each answer
async function appendEmbeddings() {
  for (let i = 0; i < qnaArray.length; i++) {
    const question = qnaArray[i].question;
    const answer = qnaArray[i].answer;
    try {
        const questionEmbedding = await openai.embeddings.create({
          model: "text-embedding-3-small",
          input: question,
          encoding_format: "float",
        });
          qnaArray[i].question_embedding = questionEmbedding.data[0].embedding;
        } catch (error) {
          console.error(`Error fetching embedding for answer ${i}:`, error);
        }
  }
    // Convert the array to a JSON string
  const data = JSON.stringify(qnaArray, null, 2);
  // Write the JSON string to a file
  fs.writeFileSync("qnaArrayWithEmbeddings.txt", data, "utf8");

  console.log("Data has been written to qnaArrayWithEmbeddings.txt");
}
appendEmbeddings();
```

The txt file looks like this. Each pair has a bunch of vectors which represent the meaning of the question.

![](/assets/images/embeddings.png){: width="700" }

And now how we can use this to answer questions? When a user asks a question, we will convert that to the same vector space. And with the help of the cosine similarity, we can find the most similar question in our dataset. Again, if you are confused, watch this!!!!!

<iframe width="560" height="315" src="https://www.youtube.com/embed/wjZofJX0v4M?si=QSNnUDNTsOiC1BsP&amp;start=902" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
