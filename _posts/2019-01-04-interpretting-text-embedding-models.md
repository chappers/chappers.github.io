---
layout: post
category : 
tags : 
tagline: 
---

_This post borrows code from [https://github.com/hiranumn/IntegratedGradients](https://github.com/hiranumn/IntegratedGradients) which in itself is based on the Integrated Gradients paper which was part of the WHI workshop at ICML 2018_

Interpretting word embedding models is fairly difficult - how do we know what words (or phrases) were an indication of why a particular instance was predicted in a certain way? 

In this post we primarily go through the code that can be used to describe this. Part of the challenge isn't so much whether it is possible or not; but rather how do we leverage existing techniques to help guide us towards something that is usable. 

Part of the challenge is that embedding layers by themselves have no concept of gradient. The reason for this is that embedding layers are merely a lookup action (that is we look up a word to get the vector), and by themselves alone do not have a gradient which we can action on. 

Instead the resolution is to attempt to derive the attribution of the embedding to the word itself. How I approached this is by creating a separate function which grabs the embedding itselve, and allowing gradient explanation methods to use this as input instead:

```py
input1 = Input(shape=(maxlen,))
embed = Embedding(max_features,
                  embedding_dims,
                  input_length=maxlen)(input1)
pool = GlobalMaxPooling1D()(embed)
dense_hidden = Dense(hidden_dims, name='hidden')(pool)
x = Activation('relu')(dense_hidden)
x = Dense(len(cats), name='prediction')(x)
pred_out = Activation('sigmoid')(x)
model = Model(inputs=[input1], outputs=[pred_out])
model.compile(loss='categorical_crossentropy',
              optimizer='adam',
              metrics=['accuracy'])
model.fit(x_train_pad, y_train,
          batch_size=batch_size,
          epochs=epochs)

model.evaluate(x_train_pad, y_train)

# get the word embedding
get_embedding = K.function([model.layers[0].input],
                           [model.layers[1].output])
x_train_idx = get_embedding([x_train_pad[idx]])[0]
#var_imp = ig.explain([x_train_pad[0]])
var_imp = ig.explain([np.max(x_train_idx, axis=0)]) # this is the mapping to the embedding layer for var_imp
```

Next, we map the provided sequence back to the name

```py
# map the text to the arrays...
col_names = [x.split() for x in tok.sequences_to_texts([x_train_pad[idx]])][0]
word_embed = x_train_idx[-len(col_names):, :]

df_embed = pd.DataFrame(word_embed, columns=["c{}".format(x) for x in range(word_embed.shape[1])])
df_embed['words'] = col_names
```

and replicate the pooling mechanism. In this scenario, we used max pooling, which makes it easy; then we can pull out the words which did surface as part of the max pooling and highlight them, as well as the contribution to the model. 

```py
# get max of every column
embed_max_colwise = np.max(word_embed, axis=0).tolist()
# subset things where the words were the max of a value...
def get_max_value_only(df, colname, val, keep_col=['words']):
    # if the value is not a max value then let's set it to zero...
    df_ = df[df[colname] == val].copy()
    keep_cols = [colname] + keep_col    
    return df_[keep_cols]

def get_colnames(df, colname, word_col):
    return '_'.join(df[df[colname] > 0][word_col].tolist())    

df_embed_ = pd.concat([get_max_value_only(df_embed, "c{}".format(col_idx), el) for col_idx, el in enumerate(embed_max_colwise)], sort=False)
df_embed_ = df_embed_.groupby(["words"]).max().reset_index()
df_embed_ = df_embed_.fillna(0)

# get column names for the embedding used...
col_names = [get_colnames(df_embed_, "c{}".format(x), 'words')+"-{}".format(x) for x in range(500)]
```

The result is that we now have what the purported column names for the max pooling operation (i.e. the embedding), and the contribution based on the embedding layer. 

From here it is fairly trivial to compute the information by essentially zipping up the information together.

```py
import pandas as pd
df = pd.DataFrame({'val':var_imp[0], 'x': col_names})
df['group'] = [1 if x > 0 else 0 for x in df['val']]
df['abs_val'] = np.abs(df['val'])

# select top 5 vals in df and then plot
df_ = df.nlargest(10, 'abs_val')
colors = {0: 'r', 1: 'b'}
df_.plot.bar(x='x', y='abs_val', color=[colors[x] for x in df_['group']])
print("Important words: "+', '.join(list(set([x.split('-')[0] for x in df_['x'].tolist()]))))
```

There we have it! How we can attribute words in a text embedding model with some measure of variable importance.
