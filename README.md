# b25 - Embedding Songs in Playlist Contexts: Adapting Word Embedding Techniques for Music Recommendation Systems

## Introduction
Training a standard word embedding model involves exposing it to a large corpus of sentences, allowing it to learn the semantic relationships between words by analyzing how they co-occur within sentence structures. This process relies on the distributional hypothesis, which suggests that words appearing in similar contexts tend to have similar meanings.

The approach in question extends this concept by drawing an analogy between sentences as collections of words and playlists as collections of songs. To construct a model for playlists, one could initially consider the song titles as the equivalent of words and the playlists as analogous to sentences. However, this direct mapping poses challenges when utilizing algorithms designed for natural language processing.

Many word embedding algorithms, such as Word2Vec, focus on local context windows—capturing the relationships between a word and its immediate neighbors. This assumption of locality doesn't directly translate to playlists, where the relationship between songs isn't necessarily dependent on their proximity or order within the playlist. Hence, adapting such algorithms for playlist modeling requires addressing the contextual difference between word co-occurrence and song co-inclusion in playlists. 

## Related work

The concept of modeling semantic relationships between entities in their context is well-established, particularly in natural language processing (NLP) [Lezama-Sánchez, 2022]. Recent advancements in this area [Chiang et al., 2020] have highlighted its applicability beyond text analysis. For instance, the principles of NLP can be adapted to playlists, where playlists provide the context, and songs represent the entities. By treating playlists as analogous to sentences and songs as words, NLP techniques like word embeddings can be repurposed, offering novel approaches for modeling relationships in music recommendation systems.

## Methodology

### Data Preprocessing
The dataset utilized in this study originates from a [Kaggle](https://www.kaggle.com/datasets/andrewmvd/spotify-playlists) repository, encompassing approximately 1GB of Spotify playlists. Preprocessing the data involved several critical steps to ensure its quality and usability for model training.

Initially, the dataset was cleaned to address common data anomalies, including outliers, missing values, and erroneous entries. Subsequently, data augmentation was performed by randomly rearranging the positions of songs within playlists, effectively expanding the dataset and enhancing its diversity. This step was motivated by the need to expose the model to a broader range of playlist structures, thereby increasing its robustness and generalization capability.

The processed dataset was then partitioned into two subsets: 90% of the data was allocated for training, while the remaining 10% was reserved for testing. This stratified split ensures a reliable evaluation of model performance while minimizing the risk of overfitting.

### Training Algorithms
#### CBOW (Continuous-Bag-of-Words)

The Continuous-Bag-of-Words (CBOW) model aims to predict a target word, known as the *center word*, based on a given context of surrounding words. This model operates under the distributional hypothesis, which suggests that words appearing in similar contexts share similar meanings. Consequently, words located closely in a text are assumed to be highly similar, whereas words that are far apart are often dissimilar in meaning.

In CBOW, the probability $P$ of predicting the center word $c$ given surrounding context words $( w_1, w_2, \ldots , w_n)$ is calculated to maximize the likelihood of the center word appearing in the context. This is formally represented as:
 
$$
P(c | w_1, w_2, \ldots, w_n) = P(c|w_1) \times P(c|w_2) \times \ldots \times P(c|w_n)
$$

where $P(c|w_i)$ represents the conditional probability of the center word given each individual context word $w_i$. The model is trained by adjusting weights to maximize this probability, leading to an embedding space that captures semantic relationships based on co-occurrence.

In a mathematical sense, the CBOW objective is to maximize the overall probability for the corpus. Given a hyper-parameter $\theta$ (which represents model parameters), the objective function is:

$$
\text{obj} = \arg \max_\theta \sum_{w \in \text{text}} \sum_{c \in \text{context}(w)} P(c|w; \theta)
$$

By training on these conditional probabilities, CBOW creates dense vector embeddings that reflect semantic similarities between words based on their contexts.


#### Skip-Gram

The Skip-Gram model, an alternative to CBOW, is designed to predict the surrounding context words based on a given center word. This approach aims to maximize the probability of observing neighboring words, given a target word. Formally, the Skip-Gram model seeks to maximize the following objective function:

$$
\sum_{t=1}^{T} \sum_{-c \le j \le c, j \ne 0} \log P(w_{t+j} | w_t)
$$

Here:
- $T$ is the total number of words in the corpus.
- $c$ denotes the window size, representing how many context words around the target word are considered.
- $P(w_{t+j} | w_t)$ is the conditional probability of a context word $w_{t+j}$ given the center word $w_t$.

In Skip-Gram, this conditional probability $P(w_{t+j} | w_t)$ is typically computed using a Softmax function, which helps assign higher probabilities to context words that appear frequently with the center word. The formula for this probability is:

$$
P(w_o | w_I) = \frac{\exp(v_{w_o}^T v_{w_I})}{\sum_{w=1}^W \exp(v_w^T v_{w_I})}
$$

where:
- $v_{w_o}$ and $v_{w_I}$ are the vector embeddings of the output (context) word $w_o$ and input (center) word $w_I$, respectively.
- $W$ is the total vocabulary size.

**Window Size Impact**: The parameter $c$, which controls the context window size, affects both accuracy and training time. Larger windows tend to capture broader context but increase computation time, as each word is paired with more context words for training.

In the context of playlist modeling, the Skip-Gram approach can be adapted by treating each song as a "word" and each playlist as a "sentence." However, unlike natural language, where the order of words conveys meaning, playlists do not always rely on the sequence of songs. Instead, Skip-Gram might capture valuable associations by treating all songs within a playlist as contextually related, without assuming that specific songs need to appear close to each other to share relevance.

#### GloVe-Inspired Co-Occurrence Embedding

Unlike CBOW and Skip-Gram, which depend on local windows of proximity to infer semantic relationships, the GloVe-like approach adopted here leverages global co-occurrence statistics drawn from entire playlists. In this adaptation, each playlist is treated as a "document," and pairs of songs that co-occur frequently within these playlists are more likely to be semantically related. To implement this, the algorithm constructs a global co-occurrence matrix, capturing how often any two songs appear together, regardless of their relative positions. Subsequently, it employs a weighted least-squares objective that aligns the inner product of song embeddings with the logarithm of their observed co-occurrence counts. Over several training epochs, the embeddings and bias terms are iteratively updated to minimize the discrepancy between the modeled and actual co-occurrences, resulting in dense vectors that reflect broader contextual relationships between songs. This method complements more localized approaches by emphasizing the overall structure of the data—enabling richer, more holistic representations that could potentially improve recommendation quality.


#### CBOS (Continuous-Bag-of-Songs)
Simpler more elegant way of calculating the embeddings - secret by now

### Model Validation
The dataset is divided into two sets: training and testing, to mitigate overfitting. The model evaluation involves selecting a song from a playlist and allowing the model to predict the next song. If the predicted song is present in the same playlist, the prediciton is marked as correct. Currently, the model is not evaluated on its ability to understand the context or sequence of songs, which represents a potential future testing phase. In essence, the model receives one song as input and suggests the next best matching song based on its learned associations.

## Results
### Model Evaluation Report
| Model              | Vector Size | Window | Min Count | Epoch | Learning Rate (Alpha) | Training Algorithm | NS Exponent | Precision@1 | Precision@3 | 
|--------------------|-------------|--------|-----------|-------|------------------------|--------------------|-------------|-------------------------------|-------------------------------|
| b25-sn-v50         | 50          | 5      | 1         | 5     | 0.025                      | CBOW               | -              | 0.3672                        | 0.3505                        |
| b25-sn-v256-a      | 256         | 5      | 1         | 5     | 0.025                      | CBOW               | -           | 0.3669                        | 0.3451                       |
| b25-sn-v256-b      | 256         | 10     | 1         | 5     | 0.025                      | CBOW               | -            | 0.4333                        | 0.3554                       |
| b25-sn-v256-c      | 256         | 20     | 1         | 5     | 0.025                      | CBOW               | -               | 0.4427                        |0.3649                       |
| b25-sn-v256-d      | 256         | 20     | 1         | 5     | 0.025                      | Skip-Gram          | 0.0             | 0.6513                        |0.5578                       |
| b25-sn-v512-a      | 512         | 100    | 1         | 5     | 0.025                      | CBOW               | -           | 0.5703                        | 0.4709                       |
| b25-sn-v512-b      | 512         | 100    | 1         | 5     | 0.025                      | Skip-Gram          | 0.0             | 0.7739                        |0.6639                       |
| b25-sn-v512-c      | 512         | inf   | 1         | 15     | 0.015                    | CBOS               | -             | 0.2196                         |N/A                       |
| b25-sn-v512-d      | 512         | inf   | 1         | 15     | 0.025                    | CBOS               | -             | 0.2362                         |N/A                       |
| b25-sn-v512-e      | 512         | inf   | 1         | 15     | 0.025                    | CBOS-d               | -             | 0.0528                        |N/A                       |
| b25-sn-v512-e*      | 512         | inf   | 1         | 15     | 0.025                    | Glove-a               | -             | 0.0095                       |N/A                       |

\* trained on 1000 Playlists

## Discussion 
**Interpretation of Results**

What do the results mean?
Unexpected findings ? 
The first iteration of the newly developed algorithm for song embedding did not align with the expectation. It was designed to fit the reduced requirements of embedding songs. More iterations needed to be done, still there is some data collected about this topic so the next one does no need to do that...

**Comparison with Prior Work**
The results of the study show that while genre-based clustering remains prevalent in playlists, deviations from this norm often arise. These deviations highlight the influence of broader cultural and social phenomena on playlist composition. Specifically, subclusters within the embeddings are more aligned with emerging trends, significant social events, and references to popular media such as movies and TV shows, rather than strictly adhering to genre boundaries. This observation aligns with existing research suggesting that user-generated content, including playlists, reflects contextual and cultural dimensions, thereby offering nuanced insights beyond traditional genre categorizations.


This research advances the application of classical vectorization algorithms by extending their usage into non-traditional domains, specifically music recommendation systems. In addition to adapting established methodologies like CBOW (Continuous Bag of Words) and Skip-Gram for embedding songs within playlists, the study introduces a novel algorithm, termed Entity Clustering (EC). This algorithm represents a contribution to the general field of vectorization by refining the representation of semantic relationships between entities.

The EC algorithm is implemented within the b25 model, which is designed to optimize the embedding of songs based on their contextual co-inclusion within playlists rather than their sequential relationships. The approach builds on the principles of contextual embedding but diverges from traditional proximity-based models to account for the unordered and association-driven nature of playlist data.

This contribution applying established algorithms in a new domain and introducing a vectorization method demonstrates the adaptability of vectorization principles and highlights their potential for cross-domain applicability. Future work aims to further refine the EC algorithm, optimize its implementation for computational efficiency, and explore its broader applicability to contexts such as dynamic media recommendations and relational analysis in complex systems.


The proposed approaches hold significant potential for application in the music industry, particularly in enhancing song recommendation systems. By leveraging these methods, personalized playlists could be tailored to align with individual user preferences, offering a highly customized listening experience. Optimization strategies may include clustering users with similar music tastes to reduce computational costs, allowing for fewer but more efficient models. Additionally, pretrained models could serve as a foundation for fine-tuning with user-specific playlists, enabling rapid adaptation to individual preferences.

The scalability of such systems provides room for innovation, such as employing transfer learning to improve recommendations across diverse user groups. Beyond user personalization, these methods could streamline the design of dynamic, mood-based playlists or assist in uncovering latent musical preferences in underrepresented genres. The interplay of optimization techniques and algorithmic design opens new pathways for exploration in both consumer and business-focused applications of music technology.

On a theoretical level, this work suggests a broader applicability of embedding-based models beyond their traditional use in natural language processing. While the focus here is on music recommendation, similar algorithms could be adapted for fields where relationships between entities need contextual representation. For instance, cryptographic applications could explore embeddings to identify patterns in data encryption processes, while interdisciplinary fields like social network analysis may utilize these techniques to model interactions and relationships.

Furthermore, embedding algorithms may find utility in biological data analysis, where they could uncover relationships in genetic sequences, protein interactions, or ecological networks. The theoretical foundation of these algorithms—rooted in capturing latent semantic relationships—can drive advancements in diverse disciplines, providing a framework for understanding complex systems. Future research should systematically explore these domains to delineate the full extent of their applicability and limitations.

Obtaining sufficient data to ensure coverage for every song within the dataset has proven to be a significant challenge. Moreover, song recommendation is inherently subjective and cannot be fully generalized due to the diverse nature of individual preferences. Consequently, the evaluation of model performance should incorporate additional metrics, such as validation through representative sampling, to improve reliability and robustness.

The computational demands of training a unique model for each user render this approach impractical, especially considering the substantial time required for model training. Furthermore, the memory complexity of each model scales as $O(n)$, where $n$ represents the number of songs incorporated in the model. This scaling limitation underscores the need for optimization strategies in both memory management and computational efficiency to handle the growth in dataset size and maintain feasible training times.

**Impact of limitations**

**Future Work**
Future work will focus on applying the insights from this study to expand the potential use cases of the proposed models, including personalized playlist recommendations, dynamic playlist generation, and cross-domain applications such as video or podcast suggestions. Additionally, optimizing the algorithms is critical to improving memory efficiency and computational performance, which will involve addressing bottlenecks in the training process, refining data handling to reduce redundancy, and implementing lower-level programming optimizations. These efforts aim to ensure scalability and robustness for handling larger datasets while maintaining or improving accuracy and speed.

**Closing Remarks**

## Conclusion 

## References

Zarlenga, M.E., Barbiero, P., Ciravegna, G., Marra, G., Giannini, F., Diligenti, M., Precioso, F., Melacci, S., Weller, A., Lio, P. and Jamnik, M., 2022, November. [Concept embedding models](https://hal.science/hal-03854550/). In NeurIPS 2022-36th Conference on Neural Information Processing Systems.

Xia, H., 2023, November. [Continuous-bag-of-words and Skip-gram for word vector training and text classification](https://iopscience.iop.org/article/10.1088/1742-6596/2634/1/012052/meta). In Journal of Physics: Conference Series (Vol. 2634, No. 1, p. 012052). IOP Publishing.

Mikolov, T., 2013. [Efficient estimation of word representations in vector space](https://www.khoury.northeastern.edu/home/vip/teach/DMcourse/4_TF_supervised/notes_slides/1301.3781.pdf). arXiv preprint arXiv:1301.3781.

Behnel, S., Bradshaw, R., Citro, C., Dalcin, L., Seljebotn, D.S. and Smith, K., 2010. [Cython: The best of both worlds](https://ieeexplore.ieee.org/abstract/document/5582062). Computing in Science & Engineering, 13(2), pp.31-39.

Johansson, R. and Johansson, R., 2015. [Code optimization. Numerical Python: A Practical Techniques Approach for Industry](https://link.springer.com/chapter/10.1007/978-1-4842-0553-2_19), pp.453-470.

Lezama-Sánchez, A.L., Tovar Vidal, M. and Reyes-Ortiz, J.A., 2022. An approach based on semantic relationship embeddings for text classification. Mathematics, 10(21), p.4161.

Chiang, H.Y., Camacho-Collados, J. and Pardos, Z., 2020, November. Understanding the source of semantic regularities in word embeddings. In Proceedings of the 24th Conference on Computational Natural Language Learning (pp. 119-131).

### General Observations and Future Steps
- **Need for Larger Test Sets**: All models should be tested against a much larger set of playlists, as testing on only 250 playlists may not provide sufficient variability and insight into model performance. Thats about 10.000 songs
- **Accuracy Potential**: It remains to be seen how high the accuracy can reach with these approaches, but further experimentation and optimization are necessary.


### Conclusion / work in progress
CBOW is better for Datasets with many fequent occuring words. Skip-Gram is better with many less frequent words ([Concluded here](https://iopscience.iop.org/article/10.1088/1742-6596/2634/1/012052/meta)). Still need to test what algorithm performes better or is it the case to create an own algorithm for embedding songs inside playlists.  Both algorithms work with some kind of context window to understand the focused word. This type of window looking should not apply to playlists because there is the whole list relevant.

## ideas
CBOW will be the main focuse here at 

Cleaning up the dataset for outliers to polish the accuracy of the trained model

It's possible that I need to implement my own model for embedding those words of the playlist to let the context window slip.

For CBOS to work i need to convert the algorithm to real c code and also optimize the operations.
--> Tried training it with the full dataset (~1GB) after 1593min of training i needed to cancel it
    --> starting with basic optimization and also possible to convert it all to c
    --> rought calculation said the training should take about 1200min...
    --> possible to throw it on some cluster idk

## notes
It is possible there is an issue with the c code or anysthing else...   
    --> 100 playlists (all ~40 Songs long) train for 1.5 min
    --> 2000 playlists still training by now 446min... 
Possible to speed up the training by about 17x by pushing an inner loop out in a way
Found an other speed up with memory management about 30x but less exact... 

Still need to test other variations of embedding algorithms to use: 
Graph Embedding Methods --> look into if it has implication
Matrix Factorization: Used in collaborative filtering to uncover latent factors in user-item interactions.
GloVe (Global Vectors): Combines global matrix factorization and local context methods to capture global co-occurrence statistics.

Designing a Custom Model:

Global Co-occurrence Focus: Shift from local context windows to capturing global co-occurrence patterns of songs within playlists.

Validation Techniques:
Employ k-fold cross-validation to ensure the model generalizes well to unseen data.
Monitor for overfitting by comparing training and validation losses.

Evaluation Metrics:

Comprehensive Metrics:
Beyond accuracy, use Precision@K, Recall@K, Mean Reciprocal Rank (MRR), and Normalized Discounted Cumulative Gain (NDCG).
A/B Testing:
If possible, conduct user studies or live A/B tests to assess recommendation quality in real-world scenarios.

The new algorithme with parts of glove and CBOW in combination is not showing realy good results... 
The training for the full size model would approx take 7 Days... 

Started training the new model with very little hope to get better results. Training will take abt 2 Days with the Optimized algorithm.

Model Training crashed after on day will need to revisisit things like datacleaning and find a new approche on traingni the model. Glove did not have good stats to start with... 

Could also implement Precision@K, Recall@K for testing

Talk about what randomguessing would be the number to set a persepctive for the results
