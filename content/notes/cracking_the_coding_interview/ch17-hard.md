---
title: Chapter 17 Hard
weight: 17
type: docs
---

## Selected Exercises

2.  Shuffle: shuffle a deck of cards perfectly.

- If the first `n-1` elements have already been shuffled, how to shuffle `n` elements?

```cpp
void shuffleArray(vector<int> &cards) {
    for (int i = 0; i < cards.size(); i++)
        swap(cards[rand(0, i)], cards[i]);
}
```

3.  Random Set: randomly generate a set of `m` integers from an array of size `n`.

- If we have selected `m` elements from the first `n-1` elements uniformly, how to decide on the `n`th element?

4.  **Missing Number: an array contains all the integers from 0 to n, except for one number which is missing. Find the missing integer.**

- Sum up the numbers?
- Analyze the missing integer bit by bit based on the oddity of `n`?
- **Convert to an equivalent problem: find the integer that occurs only once in an array where others occur exactly twice?** Add numbers 0 to n to the end of the array.

5.  **Letters and Numbers: given an array filled with letters and numbers, find the longest subarray with an equal number of letters and numbers.**

- **Track the count of letters (`#a`) and numbers (`#1`) from the start. What information could be derived about an equal subarray?** The differences of the counts (`#a - #1`) are equal at the start and at the end of the subarray.

```cpp
class Solution {
public:
    vector<string> findLongestSubarray(vector<string>& array) {
        int countDiff = 0;
        map<int, int> diffEarliestPos;
        diffEarliestPos[0] = -1;

        int endIdx = -1, maxLength = 0;
        for (int i = 0; i < array.size(); i++) {
            char c = array[i][0];
            if (c >= '0' && c <= '9') countDiff++;
            else countDiff--;

            if (diffEarliestPos.contains(countDiff)) {
                int length = i - diffEarliestPos[countDiff];
                if (length > maxLength) {
                    maxLength = length;
                    endIdx = i;
                }
            }
            else {
                diffEarliestPos[countDiff] = i;
            }
        }

        return vector<string>(array.begin() + endIdx + 1 - maxLength, array.begin() + endIdx + 1);
    }
};
```

7.  Baby Names: each year, the government releases a list of the 10000 most common baby names and their frequencies (the number of babies with that name). The only problem with this is that some names have multiple spellings. For example,"John" and ''Jon" are essentially the same name but would be listed separately in the list. Given two lists, one of names/frequencies and the other of pairs of equivalent names, write an algorithm to print a new list of the true frequency of each name.

- Merge name counts in the main table as we iterate through the synonym list?
- Graph?

8.  **Circus Tower: a circus is designing a tower routine consisting of people standing atop one anoth­er's shoulders. For practical and aesthetic reasons, each person must be both shorter and lighter than the person below him or her. Given the heights and weights of each person in the circus, write a method to compute the largest possible number of people in such a tower.**

- Dynamic programming: the tallest tower with each person as the bottom?
- **Dynamic programming: the lightest person weight that could be the bottom of a `i`-people tall tower?** If all people are sorted by heights in ascending order and by weights in descending order with same heights, the problem is equivalent to [Longest Increasing Subsequence](https://leetcode.cn/problems/longest-increasing-subsequence/) for weights. There's an `O(nlog(n))` solution.

9.  **Kth Multiple: find the kth number such that the only prime factors are 3,5, and 7.**

- Brute force: list all multiples?
- **If we have `k-1` multiples, how can we get the next multiple of 3/5/7?** Each multiple can be derived by multiplying 3/5/7 with a previous number in list. Each number `a` will eventually be used later, so we can save values of `3a, 5a, 7a` into a temporary list when we add `a`. Add the smallest element in the temporary list each time.
- **Maintain separate queues for each factor?** Designate the head of each queue as `q3, q5, q7`. If `q3` is the minimum, add `3q3,5q3,7q3` into each queue. If `q5` is the minimum, add `5q5,7q5` into Q5 and Q7, since `3q5` has already been processed (`3q5 = 3*5*q5' = 5*3*q5' = 5*q3'`, where `q3'<q5` has been added). If `q7` is the minimum, add `7q7` into Q7.

13. **Re-Space: given a dictionary (a list of strings) and the document (a string), design an algorithm to un-concatenate the document in a way that minimizes the number of unrecognized characters. Return the number of unrecognized characters.**

- Recursion: how to choose the first split location?
- How to improve time complexity by memoization?

```cpp
class Solution {
public:
    int respace(vector<string>& dictionary, string sentence) {
        unordered_set<string> dict(dictionary.begin(), dictionary.end());
        vector<int> mem(sentence.size(), -1);
        return split(dict, mem, sentence, 0);
    }

    int split(unordered_set<string> &dict, vector<int> &mem, string sentence, int start) {
        if (start >= sentence.size()) return 0;
        if (mem[start] >= 0) return mem[start];

        int min_invalid = INT_MAX;
        for (int index = start + 1; index <= sentence.size(); index++) {
            // Split before index
            string prefix = sentence.substr(start, index - start);
            int invalid = (dict.contains(prefix) ? 0 : prefix.size());
            if (invalid >= min_invalid) continue;
            
            int result = split(dict, mem, sentence, index);
            if (result + invalid < min_invalid) {
                min_invalid = result + invalid;
                if (min_invalid == 0) break;
            }
        }

        mem[start] = min_invalid;
        return min_invalid;
    }
};
```

- **Optimal solution: how to efficiently judge whether a prefix is in the dictionary?** Tries. Build a trie with the dictionary in preprocessing, and follow a path in iteration.

```cpp
class Solution {
public:
    class Trie {
    public:
        Trie* next[26] = {nullptr};
        bool isEnd = false;
        ~Trie() {
            for (int i = 0; i < 26; i++)
                if (next[i]) delete next[i];
        }
        void insert(string s) {
            Trie *current = this;
            for (int i = 0; i < s.size(); i++) {
                int c = s[i] - 'a';
                if (!current->next[c]) {
                    current->next[c] = new Trie();
                }
                current = current->next[c];
            }
            current->isEnd = true;
        }
        Trie *down(char c) {
            return next[c - 'a'];
        }
    };

    int respace(vector<string>& dictionary, string sentence) {
        Trie *trie = new Trie();
        for (const string &s : dictionary)
            trie->insert(s);

        vector<int> mem(sentence.size(), -1);
        return split(trie, mem, sentence, 0);
    }

    int split(Trie *trie, vector<int> &mem, string sentence, int start) {
        if (start >= sentence.size()) return 0;
        if (mem[start] >= 0) return mem[start];

        int min_invalid = INT_MAX;
        Trie *tmp = trie;
        for (int index = start + 1; index <= sentence.size(); index++) {
            // Split before index
            if (tmp) tmp = tmp->down(sentence[index - 1]);
            int invalid = ((tmp && tmp->isEnd) ? 0 : index - start);
            if (invalid >= min_invalid) continue;
            
            int result = split(trie, mem, sentence, index);
            if (result + invalid < min_invalid) {
                min_invalid = result + invalid;
                if (min_invalid == 0) break;
            }
        }

        mem[start] = min_invalid;
        return min_invalid;
    }
};
```

15. Longest Word: given a list of words, find the longest word made of other words in the list.

- Divide-and-conquer: iterate over all words?

```cpp
class Solution {
public:
    string longestWord(vector<string>& words) {
        sort(words.begin(), words.end(), [](const string &s1, const string &s2) {
            return (s1.size() == s2.size() ? s2.compare(s1) : s1.size() > s2.size());
        });
        map<string, bool> canBuild;
        for (const string &s : words)
            canBuild[s] = true;
        for (const string &s : words)
            if (canBuildWord(s, canBuild, true))
                return s;
        return "";
    }

    bool canBuildWord(string s, map<string, bool> &canBuild, bool original = false) {
        if (!original && canBuild.contains(s)) return canBuild[s];
        for (int index = 1; index < s.size(); index++) {
            string substring = s.substr(0, index);
            if (canBuild.contains(substring) && canBuild[substring])
                if (canBuildWord(s.substr(index, s.size() - index), canBuild))
                    return true;
        }
        canBuild[s] = false;
        return false;
    }
};
```

16. **The Masseuse: a popular masseuse receives a sequence of back-to-back appointment requests and is debating which ones to accept. She needs a break between appointments and therefore she cannot accept any adjacent requests. Given a sequence of back-to-back appoint­ment requests, find the optimal (highest total booked minutes) set the masseuse can honor. Return the number of minutes.**

- Recursion/Iteration: shall we choose `num[i]` to get the maximum result for the first `i + 1` elements?
- Memoization? Dynamic programming?
- **Do we need a `dp` table?** Only two elements before the current one is needed.

```cpp
class Solution {
public:
    int massage(vector<int>& nums) {
        int twoAway = 0;
        int oneAway = 0;
        for (int i = 0; i < nums.size(); i++) {
            int time = max(twoAway + nums[i], oneAway);
            twoAway = oneAway;
            oneAway = time;
        }
        return oneAway;
    }
};
```

17. **Multi Search: given a string band an array of smaller strings T, design a method to search b for each small string in T. Output `positions` of all strings in `smalls` that appear in big, where `positions[i]` is all positions of `smalls[i]`.**

- **Multiple patterns need to be matched against the same string. How to process the string for quicker matches?** Construct tries from all suffixes of the string (`O(b^2)` time). Each pattern can be matched in `O(k)` time.
- **Alternatively, can we build the trie with smaller strings?** We need to compare substrings from each position in the bigger string, which costs `O(bk)`. The overall elapsed time is `O(kt + bk)`, which is the optimum.

```cpp
vector<vector<int>> multiSearch(string big, vector<string>& smalls) {
    // Construct the trie with smaller strings
    Trie *trie = new Trie();
    for (int i = 0; i < smalls.size(); i++) {
        trie->insert(smalls[i], i);
    }

    // Compare substrings in the bigger string
    vector<vector<int>> result(smalls.size());
    for (int i = 0; i < big.size(); i++) {
        Trie *tmp = trie;
        for (int j = i; j < big.size(); j++) {
            tmp = tmp->down(big[j]);
            if (!tmp) break;
            if (tmp->isEnd) result[tmp->id].push_back(i);
        }
    }

    delete trie;
    return result;
}
```

18. Shortest Super-Sequence: given two arrays, one shorter (with all distinct elements) and one longer. Find the shortest subarray in the longer array that contains all the elements in the shorter array. The items can appear in any order.

- Sliding window: after a match is found, increment the left pointer, and move forward the right pointer until another is found. How does this work?
- If we record the occurrence in the longer array of each element in the shorter array in separate lists, how can we find the fitting sequences from these lists? Min-heap?

19. Missing Two: you are given an array with all the numbers from `1` to `N` appearing exactly once, except for two number that is missing. How can you find the missing number in `O(N)` time and `O(1)` space?

- How to find a unique computation that yields a result corresponding to a specific pair of missing number? Any two of `x+y`, `xy`, and `x^2+y^2`?
- If we know `x+y`, and `x<y`, can we divide the array into halves and find a single `x` in a certain half? `(x+y)/2`?

21. **Volume of Histogram: imagine a histogram (bar graph). Design an algorithm to compute the volume of water it could hold if someone poured water across the top. You can assume that each histogram bar has width 1.**

- For each pond of water, how much water is there in each row? Can we solve it with a descending monotone stack?
- For each column, how much water could it hold?
- **Can we avoid saving maxes into arrays and improve space complexity to `O(1)`?** Use two pointers `i, j`. If `h[i]<h[j]`, then we move `i` to the right next, vice versa. At position `i`, its `leftMax` has been tracked correctly in iteration, and its `rightMax` is greater than `leftMax`. Otherwise, `i` will stop at `leftMax` until `j` moves here.

22. **Word Transformer: given two words of equal length that are in a dictionary, write a method to transform one word into another word by changing only one letter at a time. The new word you get in each step must be in the dictionary.**

- **Building the whole graph could be time-consuming. How to choose the data structure so that we can look up possible words changed from a current one?** Construct a hash table, where keys are "wildcard words" (`b_ll`) and values are lists of possible words.
- **How to speed up the process of path searching?** Start from source and destination nodes simultaneously.

```cpp
class Solution {
public:
    map<string, vector<string>> dict;
    unordered_set<string> visited;

    vector<string> findLadders(string beginWord, string endWord, vector<string>& wordList) {
        for (const string &s : wordList) {
            for (int i = 0; i < s.size(); i++) {
                string tmp = s;
                tmp[i] = ' ';
                dict[tmp].push_back(s);
            }
        }

        vector<string> path;
        transform(beginWord, endWord, path);
        reverse(path.begin(), path.end());
        return path;
    }

    bool transform(string beginWord, string endWord, vector<string> &path) {
        if (beginWord == endWord) {
            path.push_back(endWord);
            return true;
        };
        visited.insert(beginWord);
        for (int i = 0; i < beginWord.size(); i++) {
            string key = beginWord;
            key[i] = ' ';
            for (const string &next : dict[key]) {
                if (!visited.contains(next)) {
                    if (transform(next, endWord, path)) {
                        path.push_back(beginWord);
                        return true;
                    }
                }
            }
        }
        return false;
    }
};
```

23. **Max Square Matrix: imagine you have a square matrix, where each cell (pixel) is either black or white. Design an algorithm to find the maximum sub-square such that all four borders are filled with black pixels.**

- Brute force: how many sub-squares are there in the matrix, and how much time does it take to check its border colors?
- **Can we cut down the time of border checking to `O(1)`?** Save the numbers of consecutive blacks on the right or below particular cells (including themselves). Fill the table just as in dynamic programming. Only four values need to be checked in this step now.

```cpp
class Solution {
public:
    vector<int> findSquare(vector<vector<int>>& matrix) {
        int n = matrix.size();

        // Consecutive blacks table
        vector<vector<int>> rightBlacks(n, vector<int>(n));
        vector<vector<int>> downBlacks(n, vector<int>(n));
        for (int i = n - 1; i >= 0; i--) {
            for (int j = n - 1; j >= 0; j--) {
                rightBlacks[i][j] = 1 - matrix[i][j];
                downBlacks[i][j] = 1 - matrix[i][j];
                if (!matrix[i][j]) {
                    if (i < n - 1) downBlacks[i][j] += downBlacks[i + 1][j];
                    if (j < n - 1) rightBlacks[i][j] += rightBlacks[i][j + 1];
                }
            }
        }

        // Enumerate all squares
        for (int size = n; size > 0; size--)
            for (int i = 0; i < n - size + 1; i++)
                for (int j = 0; j < n - size + 1; j++)
                    if (rightBlacks[i][j] >= size &&
                        downBlacks[i][j] >= size &&
                        rightBlacks[i + size - 1][j] >= size &&
                        downBlacks[i][j + size - 1] >= size)
                        return {i, j, size};
        return {};
    }
};
```

24. **Max Submatrix: given and NxM matrix of positive and negative integers, find the submatrix with the largest possible sum.**

- Brute force: how many submatrices are there, and how much time does it take to calculate the sum?
- **Can we cut down the time of computing the sum of sub-matrices?** Fill in the table of `sum[i][j]` as in dynamic programming, in which `sum[i][j]` represents the sum of the sub-matrix`M[0: i][0: j]`.
- Recall the maximum subarray problem. If we were to iterate through every contiguous sequence of rows, how can we find the set contiguous set of columns that gives the highest sum?

```cpp
vector<int> getMaxMatrix(vector<vector<int>>& matrix) {
        int n = matrix.size(), m = matrix[0].size();

        // Construct auxiliary matrix: columnSums[i][j] = sum(matrix[0:i-1][j])
        vector<vector<int>> columnSums(n + 1, vector<int>(m, 0));
        for (int i = 1; i <= n; i++)
            for (int j = 0; j < m; j++)
                columnSums[i][j] = columnSums[i - 1][j] + matrix[i - 1][j];

        // Iterate over all row sets
        int maxSum = INT_MIN;
        int maxR1 = -1, maxR2, maxC1, maxC2;
        for (int i1 = 0; i1 < n; i1++) {
            for (int i2 = i1; i2 < n; i2++) {
                int prev = 0;
                int prevStart;
                for (int j = 0; j < m; j++) {
                    int colSum = columnSums[i2 + 1][j] - columnSums[i1][j];
                    if (prev > 0) {
                        prev += colSum;
                    }
                    else {
                        prev = colSum;
                        prevStart = j;
                    }
                    if (prev > maxSum) {
                        maxSum = prev;
                        maxR1 = i1;
                        maxR2 = i2;
                        maxC1 = prevStart;
                        maxC2 = j;
                    }
                }
            }
        }
        return {maxR1, maxC1, maxR2, maxC2};
    }
```

25. **Word Rectangle: given a list of millions of words, design an algorithm to create the largest possible rectangle of letters such that every row forms a word (reading left to right) and every column forms a word (reading top to bottom). The words need not be chosen consecutively from the list but all rows must be the same length and all columns must be the same height.**

- Iterate from the maximum area possible to find the largest rectangle?
- **How to check if each row can be prefix of a word of a given length?** Construct tries for words of different lengths. If we are building a rectangle of height `h`, then only the corresponding trie is used to track paths.

```cpp
class Solution {
public:
    Trie **tries;
    vector<vector<string>> wordList;

    vector<string> maxRectangle(vector<string>& words) {
        int maxWordLength = -1;
        for (const string &s : words)
            if ((int)s.size() > maxWordLength)
                maxWordLength = (int)s.size();
        tries = new Trie *[maxWordLength]();
        wordList.resize(maxWordLength);
        for (const string &s : words)
            wordList[s.size() - 1].push_back(s);

        int maxSize = maxWordLength * maxWordLength;
        vector<string> result;
        for (int z = maxSize; z > 0; z--) {
            for (int h = 1; h <= maxWordLength; h++) {
                if (z % h == 0 && z / h <= maxWordLength) {
                    if (makeRectangle(result, h, z / h)) return result;
                }
            }
        }
        return result;
    }

    bool makeRectangle(vector<string> &result, int h, int w) {
        if (wordList[h - 1].size() == 0 || wordList[w - 1].size() == 0) return false;
        if (!tries[h - 1]) {
            tries[h - 1] = new Trie();
            for (const string &s : wordList[h - 1])
                tries[h - 1]->insert(s);
        }
        return makeRectangleHelper(result, vector<Trie *>(w, tries[h - 1]), h, w);
    }

    bool makeRectangleHelper(vector<string> &result, vector<Trie *> trieList, int h, int w) {
        if (result.size() == h) return true;
        for (const string &s : wordList[w - 1]) {
            vector<Trie *> tmp = trieList;
            bool validPrefix = true;
            for (int i = 0; i < w; i++) {
                if (!(tmp[i] = tmp[i]->down(s[i]))) {
                    validPrefix = false;
                    break;
                }
            }
            if (!validPrefix) continue;
            result.push_back(s);
            if (makeRectangleHelper(result, tmp, h, w)) return true;
            result.pop_back();
        }
        return false;
    }
};
```

26. **Sparse Similarity: the similarity of two documents (each with distinct words) is defined to be the size of the intersection divided by the size of the union. For example, if the documents consist of integers, the similarity of `{1,5,3}` and `{1,7,2,3}` is 0.4, because the intersection has size 2 and the union has size 5. We have a long list of documents (with distinct values and each with an associated ID) where the similarity is believed to be "sparse". That is, any two arbitrarily selected documents are very likely to have similarity 0. Design an algorithm that returns a list of pairs of document IDs and the associated similarity.**

- How much time does it to calculate the similarity of a pair of documents? What if the documents are sorted?
- **What would make a document similar to a specified document?** We can build a hash table that maps from a word to all documents that contain that word.
- **How can we calculate intersections more quickly?** Based on the hash table built above, if one document appears multiple times, then the number of occurrences could be equal to the intersection count.

```cpp
class Solution {
public:
    vector<string> computeSimilarities(vector<vector<int>>& docs) {
        int n = docs.size();
        map<int, vector<int>> wordToDocs;
        map<pair<int, int>, int> pairToIntersections;

        for (int i = 0; i < n; i++)
            for (int word : docs[i])
                wordToDocs[word].push_back(i);

        for (auto const &[word, docsWithWord] : wordToDocs) {
            for (int i = 0; i < docsWithWord.size(); i++) {
                for (int j = i + 1; j < docsWithWord.size(); j++) {
                    pairToIntersections[{docsWithWord[i], docsWithWord[j]}]++;
                }
            }
        }

        vector<string> result;
        stringstream stream;
        stream << std::fixed << std::setprecision(4);
        for (auto const &[docPair, intersect]: pairToIntersections) {
            double u = docs[docPair.first].size() + docs[docPair.second].size() - intersect;
            stream << docPair.first << "," << docPair.second << ": " << intersect / u + 1e-9;
            result.push_back(stream.str());
            stream.clear();
            stream.str("");
        }
        return result;
    }
};
```

