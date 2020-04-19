# How to build a simple image classification app with spoonacular API

So this is super simple and in **15 minutes** or less you will have a first working example with the spoonacular API.

Before we start, have a look at the [final result](http://htmlpreview.github.io/?https://github.com/ddsky/spoonacular-nutrition-visualizer-app/index.html) so you know what you're in for.

## Set up

This demo will use [Vue.js](https://vuejs.org/v2/guide/), for uploading an image the [Vue File Agent](https://safrazik.com/vue-file-agent/), for some fun animations [AnimeJS](https://animejs.com/), and of course the [spoonacular recipe API](https://spoonacular.com/food-api). Before you start, get a [free API key](https://spoonacular.com/food-api/pricing) for the spoonacular API and have that ready.

We will not use and SDKs but call the spoonacular API directly. If your app runs on another language than JavaScript get one of our many [SDKs](https://spoonacular.com/food-api/sdk).

## Getting started

This is a single file application so all we need is an html file. Here's the template we can start with. I leave out some styles and fields for brevity, you can always look at the[ complete file](index.html) for reference.

```html
<!DOCTYPE html>
<html>

<head>
    <!-- I'm omitting some meta fileds for brevity here -->
    <!-- Import the libraries we need -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/animejs/3.1.0/anime.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Indie+Flower&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://unpkg.com/vue-file-agent@latest/dist/vue-file-agent.css" />
    <script src="https://unpkg.com/vue-file-agent@latest/dist/vue-file-agent.umd.js"></script>
</head>

<body>

    <div id="app">
        <!-- Here's where our content will go. We let Vue handle everything in under the id #app -->
    </div>

    <script>
        var app = new Vue({
            el: '#app',
            mounted() {
            },
            data: function () {
                return {
                    // REPLACE THIS DEMO KEY, get a free key here: https://spoonacular.com/food-api
                    spoonacularApiKey: 'YOUR-SPOONACULAR-API-KEY', 
                };
            },
            computed: {
            },
            methods: {
            },
            components: {
                // import our Vue File Agent component for image uploading
                vuefileagent: VueFileAgent.VueFileAgent
            }
        });
    </script>

    <!-- Lot's of styles -->
</body>

</html>
```

If you open this file in your browser you won't see much but you should also not see any errors in your console.

Make sure you replace `YOUR-SPOONACULAR-API-KEY` with your actual key.

## Adding the components

Let's add the image uploading component directly into the `<div id="app"></div>` like so:

```html
<div id="app">
    <VueFileAgent ref="vueFileAgent" :multiple="false" :deletable="true" :uploadUrl="uploadUrl" meta="false"
                :accept="'image/*'" :maxSize="'10MB'" :maxFiles="1" @upload="onUpload($event)" compact="true"
                @select="onSelect" :helpText="'Choose images '"
                :errorText="{type: 'Invalid file type. Only images allowed',
                size: 'Files should not exceed 10MB in size',}" v-model="foodImages">
    </VueFileAgent>
</div>
```

Instead of uploading we also allow the user to click a demo image which will be sent to the API for analysis. Let's add an example image and connect it with the `analyzeWithDemo` method.

```html
<img align="right" :src="demoImageUrl" id="demoImg" @click="analyzeWithDemo" />
```

Now we need to implement the referenced methods in our Vue object in the `<script>` block. So the updated `script` section looks like this:

```html
<script>
    var app = new Vue({
        el: '#app',
        mounted() {
        },
        data: function () {
             return {
                // REPLACE THIS DEMO KEY, get a free key here: https://spoonacular.com/food-api
                spoonacularApiKey: 'YOUR-SPOONACULAR-API-KEY', 
                demoImageUrl: 'https://spoonacular.com/recipeImages/635350-240x150.jpg',
                uploadUrl: 'https://api.spoonacular.com/food/images/analyze',
                uploadHeaders: {},
                foodImages: [],
                analyzed: false,
                analyzedImage: {},
                nutrition: {}
            };
        },
        methods: {
            // this will be called after the API responds with the image analysis
            successCallback(data) {
                this.analyzedImage = data;
                this.analyzed = true;
                let self = this;
            },
            onUpload(responses) {
                this.successCallback(responses[0].data);
            },
             // we upload a file directly after it has been added to the upload container
            onSelect(event) {
                this.analyzed = false;
                this.uploadFiles();
            },
            // using the default uploader, you may use another uploader instead
            uploadFiles: function () {
                this.$refs.vueFileAgent.upload(this.uploadUrl, this.uploadHeaders, this.foodImages);
            },
            // make a direct GET request with an image URL instead of a file
            analyzeWithDemo() {
                let self = this;
                var xmlHttp = new XMLHttpRequest();
                xmlHttp.onreadystatechange = function () {
                    if (xmlHttp.readyState == 4 && xmlHttp.status == 200) {
                        self.successCallback(JSON.parse(xmlHttp.responseText));
                    }
                }
                var url = this.uploadUrl + '?apiKey=' + this.spoonacularApiKey
                    + '&imageUrl=' + encodeURIComponent(this.demoImageUrl);
                xmlHttp.open("GET", url, true);
                xmlHttp.send(null);
            }
        },
        components: {
            vuefileagent: VueFileAgent.VueFileAgent
        }
    });
</script>
```

If you run the above code it could work but you won't see anything because I omitted all the CSS and animations code above to make the code clearer. If you understand the above code snippets, you got the basics of how to work with the spoonacular API. Congratulations! Everything else is just "dressing" and making it look nice. To get the full experience, here's the complete code: 

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="UTF-8">
    <title>spoonacular Image Analyzer</title>
    <meta name="author" content="David Urbansky">
    <meta name="description" content="A spoonacular sample app.">
    <meta name="keywords" content="spoonacular, sample, app">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/animejs/3.1.0/anime.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Indie+Flower&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://unpkg.com/vue-file-agent@latest/dist/vue-file-agent.css" />
    <script src="https://unpkg.com/vue-file-agent@latest/dist/vue-file-agent.umd.js"></script>
    <link rel="shortcut icon" href="https://spoonacular.com/favicon.ico" type="image/x-icon">
    <link rel="icon" href="https://spoonacular.com/favicon.ico" type="image/x-icon">
</head>

<body>

    <div id="app">
        <h1>Upload a food picture and see what happens...</h1>

        <div id="imageChooser">
            <VueFileAgent ref="vueFileAgent" :multiple="false" :deletable="true" :uploadUrl="uploadUrl" meta="false"
                :accept="'image/*'" :maxSize="'10MB'" :maxFiles="1" @upload="onUpload($event)" compact="true"
                @select="onSelect" :helpText="'Choose images'"
                :errorText="{
              type: 'Invalid file type. Only images allowed',
              size: 'Files should not exceed 10MB in size',
            }" v-model="foodImages"></VueFileAgent>
        </div>
        <template v-if="analyzed">
            <p>
                I think this is <span class="category">{{ analyzedImage.category.name.replace('_', ' ') }}</span> -
                {{ probabilityText }}
                I only know 50 different dishes right now, here's the <a href="https://spoonacular.com/food-api/docs#Image-Classification-Categories">full list</a>.
            </p>
        </template>
        <template v-else>
            <p>&lt;- upload an image here, or see it with a <a href="#" @click="analyzeWithDemo">demo picture</a><span
                    style="position:relative;top:16px;left:5px;">⤦</span>
                <img align="right" :src="demoImageUrl" id="demoImg" @click="analyzeWithDemo" />
            </p>
        </template>
        <div style="clear:both;"></div>

        <template v-if="analyzed">
            <h1>Nutrition profile of the average <span>{{ analyzedImage.category.name.replace('_', ' ') }}</span></h1>

            <div id="calories" class="nutritionBar">
                <div></div><span>{{ nutrition.calories }} calories</span>
            </div>
            <div id="fat" class="nutritionBar">
                <div></div><span>{{ nutrition.fat }}{{ analyzedImage.nutrition.fat.unit }} fat</span>
            </div>
            <div id="protein" class="nutritionBar">
                <div></div><span>{{ nutrition.protein }}{{ analyzedImage.nutrition.protein.unit }} protein</span>
            </div>
            <div id="carbs" class="nutritionBar">
                <div></div><span>{{ nutrition.carbs }}{{ analyzedImage.nutrition.carbs.unit }} carbs</span>
            </div>

            <h1 id="hungry">Hungry now? Try one of these</h1>
            <div v-for="(r, i) in analyzedImage.recipes" :key="'recipe-' + i" class="recipe">
                <h4>{{ r.title }} </h4>
                <a :href="r.url"><img
                        :src="'https://spoonacular.com/recipeImages/' + r.id + '-240x150.' + r.imageType" /></a>
            </div>

            <!-- shameless plug -->
            <div id="spoonacular">
                powered by<br>
                <a href="https://spoonacular.com/food-api">
                    <img src="https://spoonacular.com/application/frontend/images/logo-simple-framed-green-gradient.svg"
                        alt="spoonacular logo"><br>
                    spoonacular API
                </a>
            </div>
        </template>
    </div>

    <script>
        var app = new Vue({
            el: '#app',
            mounted() {
            },
            data: function () {
                return {
                    // REPLACE THIS DEMO KEY, get a free key here: https://spoonacular.com/food-api
                    spoonacularApiKey: 'YOUR-SPOONACULAR-API-KEY', 
                    demoImageUrl: 'https://spoonacular.com/recipeImages/635350-240x150.jpg',
                    uploadUrl: 'https://api.spoonacular.com/food/images/analyze',
                    uploadHeaders: {},
                    foodImages: [],
                    analyzed: false,
                    analyzedImage: {
                        nutrition: {
                            recipesUsed: 0,
                            calories: {
                                value: 0,
                                unit: 'calories'
                            },
                            fat: {
                                value: 0,
                                unit: 'g'
                            },
                            protein: {
                                value: 0,
                                unit: 'g'
                            },
                            carbs: {
                                value: 0,
                                unit: 'g'
                            },
                        },
                        category: {
                            name: '',
                            probability: 0
                        },
                        recipes: []
                    },
                    nutrition: {
                        animationMaxWidth: 630, // maximum length for nutrition bars
                        calories100: 800, // number of calories to reach max length
                        protein100: 30, // number of grams of protein to reach max length
                        fat100: 30, // number of grams of fat to reach max length
                        carbs100: 30, // number of grams of carbs to reach max length
                        calories: 0,
                        protein: 0,
                        fat: 0,
                        carbs: 0
                    }
                };
            },
            computed: {
                probabilityText() {
                    if (this.analyzedImage.category.probability < 0.2) {
                        return 'I am really unsure about that!';
                    }
                    if (this.analyzedImage.category.probability < 0.4) {
                        return 'Maybe - maybe not though.';
                    }
                    if (this.analyzedImage.category.probability < 0.6) {
                        return 'Not really sure but looks like it.';
                    }
                    if (this.analyzedImage.category.probability < 0.8) {
                        return 'I am rather confident in that.';
                    }
                    if (this.analyzedImage.category.probability < 1) {
                        return 'I\'m almost certain!';
                    }
                }
            },
            methods: {
                animate() {
                    let self = this;
                    var nutritionTimeline = anime.timeline({
                        duration: 800,
                        easing: 'easeInOutQuad',
                    });
                    nutritionTimeline.add(
                        {
                            targets: 'div#calories div',
                            width: self.nutrition.animationMaxWidth * 
                            Math.min(1, self.analyzedImage.nutrition.calories.value / self.nutrition.calories100),
                            update: function (anim) {
                                self.nutrition.calories = 
                                Math.round(self.analyzedImage.nutrition.calories.value * anim.progress / 100);
                            },
                        }).add({
                            targets: 'div#fat div',
                            width: self.nutrition.animationMaxWidth * 
                            Math.min(1, self.analyzedImage.nutrition.fat.value / self.nutrition.fat100),
                            update: function (anim) {
                                self.nutrition.fat = 
                                Math.round(self.analyzedImage.nutrition.fat.value * anim.progress / 100);
                            },
                        }).add({
                            targets: 'div#protein div',
                            width: self.nutrition.animationMaxWidth * 
                            Math.min(1, self.analyzedImage.nutrition.protein.value / self.nutrition.protein100),
                            update: function (anim) {
                                self.nutrition.protein = 
                                Math.round(self.analyzedImage.nutrition.protein.value * anim.progress / 100);
                            },
                        }).add({
                            targets: 'div#carbs div',
                            width: self.nutrition.animationMaxWidth * 
                            Math.min(1, self.analyzedImage.nutrition.carbs.value / self.nutrition.carbs100),
                            update: function (anim) {
                                self.nutrition.carbs = 
                                Math.round(self.analyzedImage.nutrition.carbs.value * anim.progress / 100);
                            },
                        });

                    nutritionTimeline.add({
                        targets: '#hungry',
                        opacity: 1,
                        duration: 150
                    });

                    // recipes
                    nutritionTimeline.add({
                        targets: '.recipe',
                        width: 240,
                        opacity: 1,
                        rotate: '1turn',
                        delay: anime.stagger(150)
                    });

                    // logo
                    nutritionTimeline.add({
                        targets: '#spoonacular',
                        opacity: 1,
                        rotate: '1turn',
                    });
                },
                // this will be called after the API responds with the image analysis
                successCallback(data) {
                    this.analyzedImage = data;
                    this.analyzed = true;
                    let self = this;
                    Vue.nextTick(function () {
                        self.animate();
                    });
                },
                onUpload(responses) {
                    this.successCallback(responses[0].data);
                },
                // we upload a file directly after it has been added to the upload container
                onSelect(event) {
                    this.analyzed = false;
                    this.uploadFiles();
                },
                // using the default uploader, you may use another uploader instead
                uploadFiles: function () {
                    this.$refs.vueFileAgent.upload(this.uploadUrl, this.uploadHeaders, this.foodImages);
                },
                // make a direct GET request with an image URL instead of a file
                analyzeWithDemo() {
                    anime({
                        targets: '#demoImg',
                        scale: [1, 0.5],
                        opacity: [1, 0.5],
                        rotate: [0, 10, -10],
                        loop: true,
                        direction: 'alternate',
                        easing: 'easeInOutSine',
                        duration: 800
                    });

                    
                    let self = this;
                    var xmlHttp = new XMLHttpRequest();
                    xmlHttp.onreadystatechange = function () {
                        if (xmlHttp.readyState == 4 && xmlHttp.status == 200) {
                            self.successCallback(JSON.parse(xmlHttp.responseText));
                        }
                    }
                    var url = this.uploadUrl + '?apiKey=' + this.spoonacularApiKey
                     + '&imageUrl=' + encodeURIComponent(this.demoImageUrl);
                    xmlHttp.open("GET", url, true);
                    xmlHttp.send(null);
                }
            },
            components: {
                vuefileagent: VueFileAgent.VueFileAgent
            }
        });
    </script>

    <style>
        body {
            background-color: rgb(241, 255, 241);
        }

        #app {
            font-family: 'Indie Flower', cursive;
            background-color: #fff;
            max-width: 800px;
            margin-left: auto;
            margin-right: auto;
            padding: 20px;
            box-shadow: 0 1px 5px rgba(0, 0, 0, 0.2), 0 2px 2px rgba(0, 0, 0, 0.14), 0 3px 1px -2px rgba(0, 0, 0, 0.12);
            border-radius: 22px;
        }

        #demoImg {
            margin: 10px 80px 0 0;
            cursor: pointer;
            border-radius: 10px;
        }

        .vue-file-agent {
            width: 190px;
        }

        #app>h1:first-of-type {
            margin-top: 0;
        }

        h1 {
            margin-top: 60px;
        }

        h1#hungry {
            opacity: 0;
        }

        #imageChooser {
            display: inline-block;
            margin-right: 20px;
            float: left;
        }

        .recipe {
            display: inline-block;
            max-width: 240px;
            width: 50px;
            opacity: 0;
            margin-right: 12px;
            margin-bottom: 12px;
        }

        .recipe h4 {
            margin: 5px 0 5px 0;
            max-height: 50px;
            overflow: hidden;
        }

        .recipe img {
            max-width: 100%;
        }

        .nutritionBar {
            height: 30px;
            margin-bottom: 10px;
        }

        .nutritionBar div {
            height: 100%;
            width: 0;
            float: left;
            margin-right: 20px;
        }

        .nutritionBar span {
            font-weight: bold;
            font-size: 20px;
        }

        p {
            font-size: 22px;
            display: inline;
        }

        span.category {
            background-color: #29ad43;
            color: white;
            padding: 0 10px;
            border-radius: 10px;
        }

        #calories div {
            background-color: #ff5c5c;
        }

        #fat div {
            background-color: #3fcc3f;
        }

        #protein div {
            background-color: #7077ff;
        }

        #carbs div {
            background-color: #ff9400;
        }

        #spoonacular {
            margin-top: 50px;
            opacity: 0;
            margin-left: auto;
            margin-right: auto;
            width: 200px;
            text-align: center;
        }
    </style>
</body>

</html>
```

If you have questions, check our [FAQ](https://spoonacular.com/food-api/faq) or just [reach out](https://spoonacular.com/food-api/).