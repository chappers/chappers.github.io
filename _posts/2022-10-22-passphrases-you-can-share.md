
Sometimes you have to generate passphrases for sharing. Some of the word banks online just aren't...safe for work?

Its probably not worth the effort for people to curate adjectives, nouns and numbers which just fit the bill. Here is a short script that does that (refresh page to generate new passphrases)


<pre>
<p id="demo"></p>
</pre>

**How it works**

- Have seed phrases of roughly 75 animals and 75 adjectives
- Generate random 2 digit numbers
- places a `.` between the words

**What might be missing**

- Capitalisation

I still haven't figured out a sensible way to include it besides begining of the word. Perhaps you add a word seed with common names, so it alternates between

```
<name>.the.<adj>.<animal><number>
<adj>.<animal>.called.<name>
```

e.g. 

```
Albert.the.wheezy.walrus35
randiant.zebra.called.Mary86
```

Of course additional `<verb>` and `<adverb>` combinations can be added to extend the sentences. 


<script>
function generateNumber() {
return String(Math.floor(Math.random() * 10));
}

function selectRandomElement(items) {
return items[Math.floor(Math.random() * items.length)]
}

const numLen = 2;
const animals = ["albatross", "antelope", "alligator",  "bear", "blackbird", "bison", "camel", "cat", "chicken", "deer", "dog", "duck", "eagle", "elephant", "emu", "falcon", "flamingo", "frog", "giraffe", "goat", "gecko", "horse", "hummingbird", "hyena", "iguana", "ibis", "impala", "jackal", "jaguar", "jellyfish", "kangaroo", "kingfisher", "kiwi", "lion", "leopard", "lobster", "monkey", "macaw", "moose", "narwhal", "nightingale", "numbat", "owl", "octopus", "otter", "pig", "pelican", "panther", "quail", "quokka", "quoll", "rooster", "racoon", "raven", "shark", "salamander", "seal", "tiger", "turkey", "turtle", "urchin", "umbrellabird", "uguisu", "vulture", "viper", "vervet", "walrus", "whale", "wombat", "xerus", "xantus", "xeme", "yak", "yellowjacket", "yellowfin", "zebra", "zigzag", "zebu"];
const adjectives = ["adept", "adventurous", "artistic", "beloved", "breezy", "buoyant", "calm", "collected", "cool", "daring", "delightful", "dusty", "earnest", "exemplary", "exuberant", "fabulous", "fantastic", "funny", "giddy", "gusty", "glamorous", "harmless", "husky", "happy", "idle", "industrious", "impressive", "jolly", "jumpy", "jazzy", "keen", "kindly", "karmic", "leafy", "loyal", "lyrical", "magical", "maverick", "marvelous", "naive", "nostalgic", "nautical", "observant", "ornate", "overjoyed", "passionate", "puffy", "pretty", "quick", "quaint", "quirky", "radiant", "royal", "red", "special", "starry", "swift", "talented", "trusty", "tranquil", "ultimate", "utter", "utopic", "velvety", "vivid", "virtual", "wistful", "wheezy", "witty", "yellow", "youthful", "yielding", "zany", "zesty", "zippy"];

var randomAnimal = selectRandomElement(animals);
var randomAdjective = selectRandomElement(adjectives);

var randNumString = '';
for (i = 0; i < numLen; i++) {
randNumString += String(generateNumber())
}

document.getElementById("demo").innerHTML = randomAdjective + '.' + randomAnimal + randNumString;
</script>