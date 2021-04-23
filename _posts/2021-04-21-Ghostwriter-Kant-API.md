---
Title : "Ghostwriter Kant API"
layout : post
date : 2021-04-23
category : Internship
blog : true
author : dleunji
description : API 정복
---

Teachable-NLP는 코드를 따로 작성할 필요없이 **하나의 텍스트 파일**만으로 GPT-2을 튜닝할 수 있는 서비스입니다. 또한 튜닝된 모델은 서버에 자동 업로드되어, API로 모델을 활용하여 해당 텍스트를 기반으로 글을 생성하는 서비스를 개발할 수 있습니다.

그 중 저는 철학과 학부 시절 칸트가 대신 글을 써주었으면 좋겠다고 자주 상상하였습니다. 이러한 상상을 실현하고자 Teachable NLP에 칸트의 '순수이성비판'을 전처리하여 데이터를 학습시켰고, 칸트처럼 글을 쓸 수 있는 모델을 만들었습니다. 이전에 [데이터 전처리 과정](https://dleunji.github.io/blog/internship/Ghostwriter-Kant-Data/)을 상세히 설명하였기에, 이번에는 모델의 API를 활용하여 서비스를 제작한 과정을 설명하고자 합니다.

1. **모델 ID 확인**

전처리한 데이터를 입력하고, TeachableNLP을 통한 트레이닝이 끝나면 아래와 같은 화면을 보실 수 있습니다.

![_2021-04-16__6 21 07](https://user-images.githubusercontent.com/46207836/115831494-9e958180-a44c-11eb-9135-f5764188f7e2.png)

이 때, 주소로 해당 모델의 ID를 알아낼 수 있습니다. 예를 들어, GPT-2에 칸트의 '순수이성비판'을 학습시킨 후 주소창을 클릭해보면 다음과 같습니다.

<img width="621" alt="_2021-04-19__10 21 36" src="https://user-images.githubusercontent.com/46207836/115834651-6b54f180-a450-11eb-8dcc-3262a69f4385.png">

이 때, 주소의 말미에 있는 4fcddobl91jcdlqrouqh가 모델의 ID이고, ID를 통해 모델을 활용할 수 있습니다. 

**2. 모델 API**

모델이 생성된 화면 우측에 위치한 `View API` 를 누르면 모델을 활용하는 방법이 안내되어있고, `모델의 주소`와 `API`를 파악하실 수 있습니다.

제 모델 API은 [여기](https://ainize.ai/teachable-ainize/gpt2-train?branch=train/4fcddobl91jcdlqrouqh)에서 체험 가능하고, 사용 방법을 조금 더 자세히 알아보겠습니다.

Servers로 명시된 주소에 모델이 업로드 되어있습니다. 해당 서버에 Request를 보냄으로써 Response를 통해 글을 생성할 수 있습니다. **INTRO 탭**에 안내된 `curl` 명령어로 API를 테스트할 수 있지만, **API탭**에서 **별도의 코드 없이** API를 테스트 하실 수 있습니다.

<img width="999" alt="_2021-04-19__11 04 20" src="https://user-images.githubusercontent.com/46207836/115835283-106fca00-a451-11eb-8849-d37dea3128b0.png">

`Try it out!` 을 클릭하고, 직접 Request body를 작성하여 간편하게 Response를 확인할 수 있습니다.

![api](/Users/ieunji/dev/dleunji.github.io/assets/images/post/api.gif)

**POST**

![스크린샷 2021-04-22 오후 4 34 50](https://user-images.githubusercontent.com/46207836/115835936-d652f800-a451-11eb-80b4-2e13bfd7a50c.png)

- `[모델 링크]/predictions/gpt-2-en-small-finetune`에 POST 형식의 `Raw JSON Request`를 보내면 글을 생성하실 수 있습니다.

  e.g. https://train-qvbpkc5osu32fupvds7b-gpt2-train-teachable-ainize.endpoint.ainize.ai/`predictions/gpt-2-en-small-finetune`

- 헤더는 `{'Content-Type' : 'application/json; charset=utf-8'}` 입니다.

​	이 때 주의할 점은 `text가 array type`입니다.

✰✰따라서 `반드시 text는 Request 전에 Tokenizer로 인코딩` 해야합니다.✰✰

(하단 **3-B** **Backend** 코드에 **Tokenizer 사용 안내**가 있습니다.)

```json
// e.g.
{
	"text": [464, 26231,2470], //Encoded text
  "length" : 8,
  "num_samples": 2,
}
```

![스크린샷 2021-04-22 오후 4 35 00](https://user-images.githubusercontent.com/46207836/115836087-06020000-a452-11eb-96a0-9adc820e6875.png)

- **POST**

  - `[모델 링크]/predictions/gpt-2-en-small-finetune`에 POST 형식의 `Raw JSON Request`를 보내면 글을 생성하실 수 있습니다.

    e.g. https://train-qvbpkc5osu32fupvds7b-gpt2-train-teachable-ainize.endpoint.ainize.ai/`predictions/gpt-2-en-small-finetune`

  - 헤더는 `{'Content-Type' : 'application/json; charset=utf-8'}` 입니다.

  [Request Body]

  이 때 주의할 점은 `text가 array type`입니다.

  ✰✰따라서 `반드시 text는 Request 전에 Tokenizer로 인코딩` 해야합니다.✰✰

  (하단 **3-B** **Backend** 코드에 **Tokenizer 사용 안내**가 있습니다.)

  ```json
  // e.g.
  {
  	"text": [464, 26231,2470], //Encoded text
    "length" : 8,
    "num_samples": 2,
  }
  ```

  [Response]

  `Response 또한 array type`으로, 텍스트 형식으로 활용하기 위해서는 `디코딩`을 거쳐야 합니다.

  ```json
  //e.g.
  [
  	[464, 26231, 2470, 114, 55, 4982, 99, 101, 39395, 2222],
  	[464, 26231, 2470, 39948, 57, 26553, 89, 11, 482, 56]
  ]
  ```

- **GET**

  Server의 Health Check를 위한 용도입니다.

**3. 서비스에 적용**

전체적인 구조는 다음과 같습니다.

저는 모델로 단순히 칸트 식의 문장을 생성하는 것보다는 스토리와 디자인을 덧붙였습니다. 마치 칸트가 대필작가(Ghostwriter)가 되어 학생들 대신 글을 써주는 방식으로 웹페이지를 제작하여 사용자가 에세이의 운을 띄우면 칸트가 이를 기반으로 에세이를 마저 작성해주는 컨셉의 페이지입니다. 이를 위해 text를 입력 받고, 그 후 해당 text를 인코딩 후 `Fetch`를 통해 모델 API의 POST Request에 전달하는 방식으로 프로그램을 제작하였습니다.

![TeachableNLPapi 001](https://user-images.githubusercontent.com/46207836/115836353-58dbb780-a452-11eb-91af-fad68a0cb6c7.jpeg)

**A. Frontend**

아래 `id = prefix인 input에서 텍스트를 입력`받고, `search-btn이 클릭되면 submit()함수를 통해 POST request`하도록 만들었습니다.

![TeachableNLPapi2 001](https://user-images.githubusercontent.com/46207836/115836439-71e46880-a452-11eb-977b-cc7ae4284207.jpeg)

​	

```python
import requests
import json
from transformers import AutoTokenizer # For Encoding and Decoding 
autoTokenizer = AutoTokenizer.from_pretrained("gpt2-large")

@app.route('/gpt2', methods=['POST'])
def gpt2():
		# HTML에서 입력받은 폼데이터에서 텍스트 추출
    prefix = request.form['context']
		# 인코딩
    encodedText = autoTokenizer.encode(prefix)
    headers = {'Content-Type' : 'application/json; charset=utf-8'}
    data = {
        'text' : encodedText,
        'length' : 200
				# 필요 시 num_samples 설정 가능, 미설정 시 default value = 1
    }
		# 모델 주소
    url = "<https://train-avgw7n5kbmsb7wrip2a8-gpt2-train-teachable-ainize.endpoint.dev.ainize.ai/predictions/gpt-2-en-small-finetune>"
    # 해당 url로 POST request를 보낸 후, response
		response = requests.post(url, headers = headers, data = json.dumps(data))
    if response.status_code == 200:
        res = response.json()
				# 정상적인 response의 경우, response의 값을 디코딩
				# skip_special_tokens는 optional, True의 경우 디코딩 과정에서 special_token은 생략
        text = autoTokenizer.decode(res[0], skip_special_tokens=True)
				# 디코딩처리 후 Frontend로 값 전달
        return jsonify({'text' : text}), 200
    else:
        return jsonify({'fail' : 'eror'}), response.status_code
```

**4. 마무리**

NLP에 대한 지식이 전무한 상황에서 해당 서비스를 만들 수 있을지 걱정이 앞섰습니다. 하지만 단순한 API를 통해 Teachable NLP로 나만의 모델을 활용한 서비스, `Ghostwriter Kant` 를 만들 수 있었습니다. 만약 API를 모르시더라도 제 코드를 보신다면 다른 개발자들도 간단하게 Teachable NLP를 활용한 서비스를 개발하실 수 있습니다.

<img width="1439" alt="_2021-04-14__4 48 24" src="https://user-images.githubusercontent.com/46207836/115836524-8c1e4680-a452-11eb-8f70-87ef24f52a79.png">

또한 이렇게 제작한 Ghostwriter Kant를 Dockerfile과 함께 업로드 Github에 업로드한 뒤, [Ainize](https://ainize.ai/dleunji/kant?branch=ainize)에 Github에 레포지토리 주소를 입력하기만 하면 별도의 서버 비용이나 관리 없이 AI 서비스를 배포할 수 있었습니다. 제 Ghostwriter Kant 서비스를 체험해보고 싶은 분들은 [여기](https://ainize-kant-dleunji.endpoint.ainize.ai/)를 클릭해주세요.

![_2021-04-16__5 46 14](https://user-images.githubusercontent.com/46207836/115837018-1bc3f500-a453-11eb-9c47-79762e809672.png)

저와 같은 비기너들에게 NLP 모델을 튜닝하고 서버에 프로젝트를 배포하는 일은 어렵게 느껴질 겁니다. 그리고 NLP를 튜닝하는 과정에 필요한 GPU, 서버 배포 비용은 부담스럽기도 합니다.

하지만 `Teachable NLP`, `Ainize`를 활용한다면 간단한 코드 작성과 클릭만으로 프로젝트를 완성할 수 있습니다. 여러분도 여러분만의 서비스를 [포럼](https://forum.ainetwork.ai)에 개발하고 자랑해주세요! 😆



[English ver.]

Teachable NLP is a service that you can fine-tune GPT-2 models with a text file(.txt) without NLP codes.

I used [my model API](https://ainize.ai/teachable-ainize/gpt2-train?branch=train/ssasm20u83txlwa12en6) for my service, '**Ghostwriter Kant**'. My dream has finally come true! As a student majoring in Philosophy, it has been my wish for many years that Kant would write an essay on behalf of [me.To](http://me.To) realize it, I preprocessed the text extracted from “The Critique of Pure Reason” and trained it with the GPT-2 small/largemodel in Teachable NLP.

As I already explained [the training process](https://www.notion.so/Kant-The-Critique-of-Pure-Reason-a4f5c100e68642f0b07dc4fdf4ac9a3c), I will go into more detail about **using Model API for my own service** in this post.

1. **Check Model ID**

After training, The Teachable NLP screen looks like below.

![_2021-04-16__6 21 07](https://user-images.githubusercontent.com/46207836/115831494-9e958180-a44c-11eb-9135-f5764188f7e2.png)

You can find out the model ID through the url. In my case, URL was like below. 

<img width="621" alt="_2021-04-19__10 21 36" src="https://user-images.githubusercontent.com/46207836/115834651-6b54f180-a450-11eb-8dcc-3262a69f4385.png">

The modle ID is a series of letters in the end of URL, that is `4fcddobl91jcdlqrouqh` .

**2. Model API**

If you click the  `View API` , the `API` guide appears. You can experience my Model API  [here](https://ainize.ai/teachable-ainize/gpt2-train?branch=train/4fcddobl91jcdlqrouqh). Let me show you more detailed guide.

The fine-tuned model is uploaded to the Servers of the url. You can get the generated text through `response` by sending `POST Request`. It is possible to test API not only using `curl` command as it is written in **INTRO Tab**, however, but also using `swagger` in **API Tab** without any codes.

<img width="999" alt="_2021-04-19__11 04 20" src="https://user-images.githubusercontent.com/46207836/115835283-106fca00-a451-11eb-8849-d37dea3128b0.png">

Just click `Try it out!` , and fill out Request body for yourself. Then you can check out Response easily.

![api](/Users/ieunji/dev/dleunji.github.io/assets/images/post/api.gif)

 **POST**

![스크린샷 2021-04-22 오후 4 34 50](https://user-images.githubusercontent.com/46207836/115835936-d652f800-a451-11eb-80b4-2e13bfd7a50c.png)

​	

- Just send  `Raw JSON Request`  in POST method to `[Model url]/predictions/gpt-2-en-small-finetune`

  e.g. https://train-qvbpkc5osu32fupvds7b-gpt2-train-teachable-ainize.endpoint.ainize.ai/`predictions/gpt-2-en-small-finetune`

- The header is  `{'Content-Type' : 'application/json; charset=utf-8'}` .

Be careful to the type of text is `array type` !

✰✰So you should `encode the text before the Request`.✰✰

(It is specifically guided to **3-B** **Backend** codes using **Tokenizer**.)

![스크린샷 2021-04-23 오후 4 49 25](https://user-images.githubusercontent.com/46207836/115837766-e7046d80-a453-11eb-81ca-0469b431f83d.png)

- **POST**

  [Request Body]

  - Just send  `Raw JSON Request`  in POST method to `[Model url]/predictions/gpt-2-en-small-finetune`

    e.g. https://train-qvbpkc5osu32fupvds7b-gpt2-train-teachable-ainize.endpoint.ainize.ai/`predictions/gpt-2-en-small-finetune`

  - The header is  `{'Content-Type' : 'application/json; charset=utf-8'}` .

  Be careful to the type of text is `array type` !

  ✰✰So you should `encode the text before the Request`.✰✰

  (It is specifically guided to **3-B** **Backend** codes using **Tokenizer**.)

  ```json
  // e.g.
  {
  	"text": [464, 26231,2470], //Encoded text
    "length" : 8,
    "num_samples": 2,
  }
  ```

  [Response]

  `Response is also array type`, so you should `decode the array to text`.

  ```json
  //e.g.
  [
  	[464, 26231, 2470, 114, 55, 4982, 99, 101, 39395, 2222],
  	[464, 26231, 2470, 39948, 57, 26553, 89, 11, 482, 56]
  ]
  ```

- **GET**

  It is used for Health Check of server.

**3. Apply to my own service**

It is more understandable with diagram of the API flow. I added story and design to make more interesting service, not just generating the Kantian sentences. As if Kant becomes a ghostwriter and writes an essay in place of students, I made Web page and if user let him know the beginning of the essay, he picks up where we said.

So I made the program **get** some input from users, **encode** the text, **fetch** the generated text using POST Request and finally **decode** the text from response.

![TeachableNLPapi 001](https://user-images.githubusercontent.com/46207836/115836353-58dbb780-a452-11eb-91af-fad68a0cb6c7.jpeg)

**A. Frontend**

Get text through `input of which id is 'prefix'`, and if `search-btn is clicked, POST request occurs through submit()` .

![TeachableNLPapi2 001](https://user-images.githubusercontent.com/46207836/115836439-71e46880-a452-11eb-977b-cc7ae4284207.jpeg)

It is coded using `Promise` of Javascript.

```jsx
function submit(){
	var prefix = document.getElementById("prefix");
	var prefix_value = prefix.value;
	/* If there input box is blank */
	if (prefix_value == ""){
	    alert('Please let me know how to get started!');
	    return;
	}
	var formData = new FormData();
	/* To append the prefix to form data, and make  {"context" : prefix_value}. */
	formData.append("context", prefix_value);
	/* To fetch the form to /gpt2, go through app.py, send Request, and then 
get response from app.py*/
	fetch(
	    "/gpt2", 
	    {
	        method: "POST",
	        body : formData
	    }
	)
	/* The Reponse returned from request is parameter */ 
	/* The request is encoded, and decoded like {'text: "..."} in app.py */
	.then(response => {
	    if(response.status == 200){
	        return response;
	    }
	    else if(response.status == 400){
	        throw Error("Failed1");
	    }
	    else {
	        throw Error("Failed2");
	    }
	})
	.then(response => {
	    res = response.json()
	    return res
	})
	/* To change HTML dynamically */
	.then(response => {
	    var text = response['text'] + "...<br>Now it's your turn!<br><br><br>";
	    document.getElementsByClassName("essay")[0].innerHTML = text;
	})
	/* To handle the errors */
	.catch(e => {
	    var result = document.getElementsByClassName("essay")[0];
	    result.textContent = e;
	})
}
```

**B. Backend**

`Flask`is used for tokenizing .

```python
import requests
import json
from transformers import AutoTokenizer # For Encoding and Decoding 
autoTokenizer = AutoTokenizer.from_pretrained("gpt2-large")

@app.route('/gpt2', methods=['POST'])
def gpt2():
		# Extract text from form data sent from HTML
    prefix = request.form['context']
		# Encoding
    encodedText = autoTokenizer.encode(prefix)
    headers = {'Content-Type' : 'application/json; charset=utf-8'}
    data = {
        'text' : encodedText,
        'length' : 200
				# 'num_samples' key is optional, its default value is 1.
		}
		# The server url
    url = "<https://train-avgw7n5kbmsb7wrip2a8-gpt2-train-teachable-ainize.endpoint.dev.ainize.ai/predictions/gpt-2-en-small-finetune>"
    # To send POST request to url,and get response
		response = requests.post(url, headers = headers, data = json.dumps(data))
    if response.status_code == 200:
        res = response.json()
				# To Decode the value of response if response is valid
				# 'skip_special_tokens' is optional. If it is True, special_token is excepted
        text = autoTokenizer.decode(res[0], skip_special_tokens=True)
				# After decoding, deliver the text to frontend.
        return jsonify({'text' : text}), 200
    else:
        return jsonify({'fail' : 'eror'}), response.status_code
```

**4. Wrap up**

I worried if I could create the `Ghostwriter Kant` in the absence of knowledge of NLP. However, through a simple API of Teachable NLP, I was able to create it with my own model.  Even if you don't know the API, other beginners can easily make service using Teachable NLP refering to my codes.

<img width="1439" alt="_2021-04-14__4 48 24" src="https://user-images.githubusercontent.com/46207836/115836524-8c1e4680-a452-11eb-8f70-87ef24f52a79.png">

Moreover, if you upload the project with `Dockerfile` in Github and just fill out the url of repository of Github in [Ainize](https://ainize.ai/dleunji/kant?branch=ainize), you can serve your own AI service without cost of server.

Anyone who wants to experience my `Ghostwriter Kant` service, please click [here](https://ainize-kant-dleunji.endpoint.ainize.ai/)!

![_2021-04-16__5 46 14](https://user-images.githubusercontent.com/46207836/115837018-1bc3f500-a453-11eb-9c47-79762e809672.png)

It may be difficult for beginners to fine-tune the model and serve the project. Also the GPU needed to fine-tuning, and the cost of server is more intolerably burdensome.

But `Teachable NLP`, `Ainize` solve the problems. You just write some codes and clicks, you can complete your own project! How about boasting of your service in the [forum](https://forum.ainetwork.ai)! 😆



