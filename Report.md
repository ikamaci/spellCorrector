# CMPE 493 Information Retrieval - Spelling Error Corrector
##### Contributor Name: İbrahim Kamacı
##### StudentID: 2015400141


# Implementation
  
  In this project **Python 3.7** and **Jupyter Notebook** is used for implementation.\
  In order to implement this project **3** Python libraries were used. These are:

  ```json``` : Used for reading and writing files. For example ```corpus.txt```
  ```string``` : Used for removing punctuations from the corpus. It could be done by manually  but in ```string.punctuation``` contains all of them. 
  ```numpy``` : Used only for declaring two dimentional array with zeros.

## 1. Corpus

- ### 1.1 Tokenization
	In order to create a dictionary first we need to tokenize corpus file by performing couple of operations.\
     First the program removes all punctuations from the corpus file by performing following code fragment. To perform 		such operation python string library is used. 
    ```python
 	#Get rid of all punctuations
	corpus = corpus.translate(str.maketrans(string.punctuation, ' '*len(string.punctuation)))
    ```
    Secondly, case folding is done by the code below
     ```python
 	#Case folding 
	corpus = corpus.lower()
    ```
    Lastly split by white-space operation is performed by following code fragment.
    
    ```python
 	corpusArr = corpus.split()
    ```
    
- ### 1.2 Dictionary
	Necessary steps were done in previous part to construct a token dictionary from corpus file. Now we need to create a 	token - frequency dictionary by traversing ```corpusArr``` varible and count number of occurences of each token.
   
   ```python
   
    for token in corpusArr:
    	if token not in corpus_tokenized:
        	corpus_tokenized[token] = 1
      	else:
        	corpus_tokenized[token] +=1
    ```
## 2. Damerau-Levenshtein Edit Distance

In this project calculating  Damerau-Levenshtein edit distance for 2 words is essential.\ Both operations that is to 	construct 4 different confusion matrices and to select correct-word candidates which their edit distances to misspelled-word equals 1, we need to edit distance calculator function.\ Edit distance of two words is calculated via dynamic programming fashion just like we have learnt from the lecture. In such function edit distance of two word can be calculated and furthermore, by back-tracking the table which was constructed while calculating edit distance, all ```del[x,y]``` , ```ins[x,y]```, ```sub[x,y]``` and ```trans[x,y]``` operations can be stored to construct 4 confusion matrices. This is because back-tracking is not alwasy necessary and it increases the execution time, it is designed to be optional operation by passing a boolean ``` onlyDistance``` Damerau-Levenshtein edit distance calculation is performed via code fragment below.

```python

    def LevDistance(word1,word2,onlyDistance):
        matrix = []
        word1 = "$" + word1
        word2 = "$"+ word2
        nextElem = (len(word2)-1,len(word1)-1)
        operations = []
        channelOp = []
        distance = 0
        matrix = numpy.zeros((len(word2),len(word1)))
        for k in range(0,len(word1)):
            matrix[0,k] = k
        for l in range(0,len(word2)):
            matrix[l,0] = l    
        for i in range(1,len(word2)):
            for j in range(1,len(word1)):
                if word1[j] == word2[i]:
                    matrix[i,j] = min(matrix[i-1,j-1],matrix[i-1,j]+1,matrix[i,j-1]+1)
                else:
                    if word1[j] == word2[i-1] and word1[j-1] == word2[i] and i !=1 and j != 1:
                        matrix[i,j] = min(matrix[i-1,j-1]+1,matrix[i-1,j]+1,matrix[i,j-1]+1,matrix[i-2,j-2]+1)    
                    else:
                        matrix[i,j] = min(matrix[i-1,j-1]+1,matrix[i-1,j]+1,matrix[i,j-1]+1)
        if not onlyDistance:
            for i in range(len(word2)-1,-1,-1):
                for j in range(len(word1)-1,-1,-1):
                    if nextElem[0] == i and nextElem[1] ==j:
                        if word1[j] == word2[i]:
                            if matrix[i,j] == matrix[i-1,j-1]:
                                nextElem = (i-1,j-1)
                                operations.append(["cpy",word1[j],word2[i]])
                            elif matrix[i,j] == matrix[i,j-1]+1:
                                nextElem = (i,j-1)
                                operations.append(["ins","*",word1[j]])
                                channelOp.append(["del",word1[j-1],word1[j]])
                                distance = distance + 1
                            elif matrix[i,j] == matrix[i-1,j]+1:
                                nnextElem = (i-1,j)
                                operations.append(["del",word2[i],"*"])
                                channelOp.append(["ins",word1[j-1],word2[i]])
                                distance = distance + 1
                        else:
                            if word1[j] == word2[i-1] and word1[j-1] == word2[i] and i > 1 and j > 1:
                                nextElem = (i-2,j-2)
                                operations.append(["trans",word1[j],word2[i]])
                                channelOp.append(["trans",word2[i],word1[j]])
                                distance = distance + 1
                            if matrix[i,j] == matrix[i-1,j-1]+1:
                                nextElem = (i-1,j-1)
                                operations.append(["sub",word2[i],word1[j]])
                                channelOp.append(["sub",word1[j],word2[i]])
                                distance = distance + 1
                            elif matrix[i,j] == matrix[i,j-1]+1:
                                nextElem = (i,j-1)
                                operations.append(["ins","*",word1[j]])
                                channelOp.append(["del",word1[j-1],word1[j]])
                            elif matrix[i,j] == matrix[i-1,j]+1:
                                nextElem = (i-1,j)
                                operations.append(["del",word2[i],"*"])
                                channelOp.append(["ins",word1[j],word2[i]])
        distance = matrix[len(word2)-1,len(word1)-1]
        return (matrix,operations,channelOp,distance)
  ```
  ```LevDistance``` method takes 3 arguments ```word1```, ```word2``` and ```onlyDistance```, returns ```matrix```, ```operations``` ,```channelOp``` and ```distance``` 
  First two variables primarly designed for testing issues.

## 3. Confusion Matrices
  - ### 3.1 Data Structure
    Confusion matrices are stored as dictionary of dictionary. That means a dictionary which its keys equals to all english letters and ```$``` sign in the first layer and all keys has ,again, all english letters and ```$```. Illustration of confusion matrix varible:
  
  ```python
    confusionMatrix = {

      '$': {
        '$': 0,
        'a': 2,
        'b': 2
         .
         .
         .
      },
      'a': {
        '$': 0,
        'a': 0,
        'b': 43,
         .
         .
         .
      },
      .
      .
      .
    }

  ```   
  For example, If ```sub['c','k'] ``` is needed, then it can be reached via ```subConfusionMatrix['c']['k']```
 
 - ### 3.2 Construction

    In order to cunstruct confusion matrices, we need to read ```spell-errors.txt``` file. Then, read each lines that contain correct and misspelled versions of words. For each correct-misspelled
  word pair, edit operations should be extracted and stored into proper confusion matrix. Following function performs such operation.

  ```python

    def readErrorFile(filePath):
      alphabet = ['$','a','b','c','d','e','f','g','h','i','j','k','l','m','n','o','p','q','r','s','t','u','v','w','x','y','z']
      subMatrix = dict.fromkeys(alphabet)
      delMatrix = dict.fromkeys(alphabet)
      transMatrix = dict.fromkeys(alphabet)
      insMatrix = dict.fromkeys(alphabet)
      for letter in alphabet:
          subMatrix[letter] = dict.fromkeys(alphabet,0)
          subMatrix[letter] = dict.fromkeys(alphabet,0)
          delMatrix[letter] = dict.fromkeys(alphabet,0)
          transMatrix[letter] = dict.fromkeys(alphabet,0)
          insMatrix[letter] = dict.fromkeys(alphabet,0)
      channelOperationAll = []
      spellFile = open(filePath,'r')
      spellError = spellFile.read()
      spellError = spellError.lower()
      translator = str.maketrans('', '', "!\"#$%&'()+-./;<=>?@[\]^_`{|} ~")
      spellError = spellError.translate(translator)
      lines = spellError.split('\n')
      for line in lines:
          lineArr = line.split(':')
          correct = lineArr[0]
          wrongArr = lineArr[1]
          wrongArr = wrongArr.split(',')
          for misspell in wrongArr:
              misspell = misspell.strip()
              index = misspell.find("*")
              if index == -1:
                  matrix,op,channelOp,distance=(LevDistance(correct, misspell,False))
                  for operation in channelOp:
                      if operation[0] == 'ins':
                          insMatrix[operation[1]][operation[2]] += 1
                      elif operation[0] == 'sub':
                           subMatrix[operation[1]][operation[2]] += 1
                      elif operation[0] == 'trans':
                           transMatrix[operation[1]][operation[2]] += 1
                      elif operation[0] == 'del':
                           delMatrix[operation[1]][operation[2]] += 1
              else:
                  missSpellArr = misspell.split('*')
                  count = int(missSpellArr[1])
                  misspell= missSpellArr[0]
                  matrix,op,channelOp,distance=(LevDistance(correct, misspell,False))
                  for operation in channelOp:
                      if operation[0] == 'ins':
                          insMatrix[operation[1]][operation[2]] += count
                      elif operation[0] == 'sub':
                           subMatrix[operation[1]][operation[2]] += count
                      elif operation[0] == 'trans':
                           transMatrix[operation[1]][operation[2]] += count
                      elif operation[0] == 'del':
                           delMatrix[operation[1]][operation[2]] += count
      return (subMatrix,delMatrix,transMatrix,insMatrix)

  ```      

## 4. Spelling Error Corrector

  Spelling error corrector performs two main steps. First selecting a proper correct version candidate of the misspelled word. Second selecting one correct word candidate among all candidates.
  
  - ### 4.1 Selecting Candidates

    In spelling error corrector first, edit distances between misspelled word and each token dictionary items should be calculated. Because in this project we need to select correct-word 
    candidates that its edit distance to misspelled word equals 1. Iterating over dictionary and stores candidates.\
    > **Note:** *For increasing execution time, the program checks only tokens that their length difference from the lenght of misspelled word by 1. Because if absolute value of length differences is greater than or equal to 2, then edit distance could never be 1. This check improves execution time dramatically.*

    ```python
      for line in lines:
        candidates = []
        correct = ""
        for token in corpus_tokenized:
            if abs(len(token) - len(line)) < 2:
                matrix,operations,channelOp,distance = LevDistance(token,line,True)
                if distance < 2:
                    candidates.append(token)
    ```
- ### 4.1 Selecting Correct Word

  If number of candidates are 1 than the correct word it is. However, If number of candidates are more than one, we need to maximize the probibility ```P(x|w)P(w)```\
  In this step, program iterates over candidates and calculate ```P(x|w)P(w)``` for each by looking dictionary for ```P(w)``` and by looking proper confusion matrix by ```P(x|w)```\
  Then selects the one that maximizes the probibility.

  ```python
      if len(candidates) > 1:
            P_x_w = 0
            for currentCandidate in candidates:
                matrix,operations,operation,distance = LevDistance(token,line,False)
                if operation[0][0] == 'ins':
                    if not smooth:
                        prob = (insConfusionMatrix[operation[0][1]][operation[0][2]]/corpus_tokenized[currentCandidate]) * (corpus_tokenized[currentCandidate]/corpusSize)
                    else: 
                        prob = (insConfusionMatrix[operation[0][1]][operation[0][2]]+1)/(corpus_tokenized[currentCandidate]+27) * (corpus_tokenized[currentCandidate]/corpusSize)
                    if prob > P_x_w:
                        correct = currentCandidate
                elif operation[0][0] == 'sub':
                    if not smooth:
                        prob = (subConfusionMatrix[operation[0][1]][operation[0][2]]/corpus_tokenized[currentCandidate]) * (corpus_tokenized[currentCandidate]/corpusSize)
                    else:
                        prob = (subConfusionMatrix[operation[0][1]][operation[0][2]]+1)/(corpus_tokenized[currentCandidate]+27) * (corpus_tokenized[currentCandidate]/corpusSize)
                    if prob > P_x_w:
                        correct = currentCandidate
                elif operation[0][0] == 'trans':
                    if not smooth:
                        prob = (transConfusionMatrix[operation[0][1]][operation[0][2]]/corpus_tokenized[currentCandidate]) * (corpus_tokenized[currentCandidate]/corpusSize)
                    else: 
                        prob = (transConfusionMatrix[operation[0][1]][operation[0][2]]+1)/(corpus_tokenized[currentCandidate]+27) * (corpus_tokenized[currentCandidate]/corpusSize)
                    if prob > P_x_w:
                        correct = currentCandidate
                elif operation[0][0] == 'del':
                    if not smooth:
                        prob = (delConfusionMatrix[operation[0][1]][operation[0][2]]/corpus_tokenized[currentCandidate]) * (corpus_tokenized[currentCandidate]/corpusSize)
                    else: 
                        prob = (delConfusionMatrix[operation[0][1]][operation[0][2]]+1)/(corpus_tokenized[currentCandidate]+27) * (corpus_tokenized[currentCandidate]/corpusSize)
                    if prob > P_x_w:
                        correct = currentCandidate
        elif len(candidates) == 1:
            correct = candidates[0]

  ```

# Report 

## 1. Confusion Matrices 

  - ### 1.1 Trans Matrix
  
  
|Letter| " "  |a  |b  |c  |d  |e  |f  |g  |h  |I  |j  |k  |l  |m  |n  |o  |p  |q  |r  |s  |t  |u  |v  |w  |x  |y  |z  |
|------|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|" "   |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |
|a     |0  |0  |31 |17 |0  |7  |0  |3  |0  |56 |0  |7  |28 |12 |64 |3  |2  |0  |71 |37 |47 |17 |2  |2  |0  |5  |0  |
|b     |0  |1  |0  |0  |0  |2  |0  |0  |0  |14 |0  |0  |8  |0  |0  |10 |0  |0  |1  |0  |0  |0  |0  |0  |0  |0  |0  |
|c     |0  |10 |0  |0  |0  |43 |0  |0  |7  |123|0  |1  |4  |0  |0  |8  |0  |0  |1  |0  |4  |7  |0  |0  |0  |4  |0  |
|d     |0  |4  |0  |0  |0  |30 |0  |10 |0  |22 |0  |0  |1  |0  |0  |3  |0  |0  |2  |0  |0  |15 |0  |0  |0  |1  |0  |
|e     |0  |28 |2  |18 |96 |0  |30 |7  |0  |83 |1  |1  |119|66 |93 |5  |13 |0  |183|114|45 |21 |1  |3  |6  |5  |5  |
|f     |0  |2  |0  |0  |0  |19 |0  |0  |0  |28 |0  |0  |0  |0  |0  |6  |0  |0  |0  |0  |1  |2  |0  |0  |0  |0  |0  |
|g     |0  |23 |0  |0  |0  |24 |0  |0  |2  |27 |0  |0  |0  |0  |14 |2  |0  |0  |2  |0  |0  |6  |0  |0  |0  |1  |0  |
|h     |0  |9  |0  |0  |0  |10 |0  |0  |0  |4  |0  |0  |0  |0  |0  |6  |0  |0  |0  |0  |33 |1  |0  |0  |0  |0  |0  |
|I     |0  |36 |6  |55 |11 |132|6  |5  |0  |0  |0  |0  |32 |27 |57 |31 |6  |0  |28 |88 |46 |5  |9  |0  |0  |0  |1  |
|j     |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |
|k     |0  |0  |0  |0  |0  |5  |0  |0  |0  |0  |0  |0  |0  |0  |2  |0  |0  |0  |0  |2  |0  |0  |0  |0  |0  |0  |0  |
|l     |0  |18 |0  |0  |9  |155|0  |1  |0  |66 |0  |1  |0  |0  |0  |37 |3  |0  |0  |4  |5  |9  |1  |0  |0  |5  |0  |
|m     |0  |24 |1  |0  |0  |72 |0  |0  |0  |46 |0  |0  |0  |0  |2  |13 |0  |0  |0  |2  |0  |5  |0  |0  |0  |0  |0  |
|n     |0  |64 |0  |1  |8  |143|0  |7  |0  |113|0  |4  |2  |1  |0  |35 |0  |0  |0  |3  |5  |8  |0  |0  |0  |1  |0  |
|o     |0  |7  |1  |1  |16 |8  |1  |3  |0  |14 |0  |0  |28 |14 |8  |0  |12 |0  |59 |5  |12 |17 |1  |7  |0  |1  |0  |
|p     |0  |11 |0  |0  |0  |28 |0  |0  |2  |1  |0  |0  |5  |0  |0  |11 |0  |0  |6  |4  |0  |1  |0  |0  |0  |0  |0  |
|q     |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |2  |0  |0  |0  |0  |0  |
|r     |0  |101|0  |2  |1  |339|0  |1  |0  |50 |0  |2  |0  |2  |1  |41 |0  |0  |0  |5  |3  |14 |0  |0  |0  |4  |0  |
|s     |0  |2  |0  |7  |1  |84 |1  |0  |0  |93 |0  |0  |0  |2  |0  |22 |1  |0  |0  |0  |18 |17 |0  |4  |0  |10 |0  |
|t     |0  |33 |0  |9  |0  |147|0  |0  |23 |106|0  |0  |8  |0  |0  |5  |0  |0  |9  |27 |0  |8  |0  |0  |0  |0  |0  |
|u     |0  |44 |1  |4  |10 |7  |0  |0  |0  |16 |0  |0  |8  |5  |8  |7  |1  |4  |21 |13 |5  |0  |0  |0  |0  |0  |0  |
|v     |0  |14 |0  |0  |0  |29 |0  |0  |0  |10 |0  |0  |0  |0  |0  |1  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |
|w     |0  |2  |0  |0  |0  |4  |0  |0  |7  |2  |0  |0  |1  |0  |5  |6  |0  |0  |1  |0  |0  |0  |0  |0  |0  |0  |0  |
|x     |0  |0  |0  |1  |0  |1  |0  |0  |1  |2  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |1  |0  |0  |0  |0  |0  |0  |
|y     |0  |1  |0  |17 |0  |8  |0  |0  |0  |2  |0  |0  |1  |0  |0  |0  |0  |0  |0  |6  |0  |0  |0  |0  |0  |0  |0  |
|z     |0  |0  |0  |0  |0  |2  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |

  - ### 1.2 Sub Matrix

|Letter |" "  |a   |b  |c  |d  |e   |f  |g  |h  |I   |j  |k  |l  |m  |n  |o  |p  |q  |r  |s   |t  |u  |v  |w  |x  |y  |z  |
|------|---|----|---|---|---|----|---|---|---|----|---|---|---|---|---|---|---|---|---|----|---|---|---|---|---|---|---|
|" "   |0  |1   |0  |0  |4  |1   |0  |0  |3  |0   |0  |0  |0  |1  |3  |0  |0  |0  |5  |3   |4  |0  |3  |1  |0  |1  |1  |
|a     |1  |0   |24 |103|54 |1727|27 |27 |74 |750 |1  |32 |197|37 |138|796|29 |3  |419|142 |99 |369|17 |16 |3  |36 |7  |
|b     |1  |15  |0  |2  |85 |42  |5  |6  |4  |11  |0  |1  |23 |12 |14 |8  |86 |0  |18 |2   |13 |15 |19 |2  |0  |1  |0  |
|c     |2  |130 |4  |0  |31 |266 |23 |68 |66 |98  |0  |163|64 |16 |73 |63 |34 |52 |77 |1256|332|126|20 |8  |85 |21 |14 |
|d     |2  |74  |90 |18 |0  |165 |6  |107|28 |40  |15 |16 |81 |20 |85 |22 |15 |0  |78 |87  |428|12 |10 |8  |1  |33 |2  |
|e     |4  |1736|37 |93 |174|0   |92 |77 |122|1733|7  |74 |385|75 |245|506|48 |2  |367|636 |466|486|31 |37 |6  |291|13 |
|f     |1  |34  |5  |24 |9  |78  |0  |7  |68 |15  |0  |0  |18 |6  |19 |17 |43 |2  |78 |24  |48 |10 |118|4  |7  |6  |0  |
|g     |0  |69  |9  |121|110|129 |23 |0  |27 |38  |34 |37 |35 |12 |66 |39 |9  |19 |57 |40  |50 |21 |10 |3  |1  |41 |8  |
|h     |4  |81  |5  |55 |27 |134 |180|25 |0  |70  |4  |86 |54 |8  |44 |74 |17 |3  |84 |51  |149|43 |24 |27 |3  |15 |0  |
|I     |4  |1150|14 |157|61 |2274|47 |51 |270|0   |2  |10 |221|70 |252|348|28 |5  |320|296 |209|322|19 |11 |4  |242|21 |
|j     |0  |2   |0  |8  |56 |3   |0  |75 |1  |0   |0  |0  |3  |2  |0  |2  |1  |1  |4  |0   |2  |2  |0  |0  |0  |4  |0  |
|k     |0  |9   |1  |97 |7  |44  |1  |23 |15 |3   |0  |0  |8  |2  |11 |16 |0  |2  |10 |8   |24 |2  |0  |4  |1  |1  |0  |
|l     |1  |293 |15 |61 |55 |444 |28 |18 |37 |188 |1  |20 |0  |15 |108|145|17 |5  |266|49  |163|128|18 |16 |2  |36 |3  |
|m     |2  |84  |13 |19 |26 |155 |6  |9  |15 |61  |0  |3  |22 |0  |575|62 |20 |0  |45 |18  |37 |36 |6  |9  |5  |13 |1  |
|n     |5  |167 |8  |60 |107|376 |15 |43 |78 |167 |0  |10 |86 |401|0  |121|22 |2  |150|74  |170|105|12 |20 |9  |21 |1  |
|o     |4  |692 |11 |84 |37 |645 |9  |35 |65 |292 |4  |31 |104|28 |133|0  |20 |4  |178|61  |48 |375|12 |22 |4  |21 |3  |
|p     |1  |59  |69 |41 |7  |76  |59 |7  |28 |62  |0  |15 |13 |21 |35 |22 |0  |0  |40 |37  |83 |18 |7  |0  |4  |11 |0  |
|q     |0  |7   |0  |79 |1  |7   |0  |24 |5  |5   |0  |22 |1  |1  |1  |9  |7  |0  |1  |2   |8  |1  |0  |0  |0  |0  |3  |
|r     |2  |261 |14 |42 |53 |405 |22 |30 |74 |188 |2  |12 |117|27 |146|147|29 |1  |0  |78  |96 |163|16 |36 |3  |51 |2  |
|s     |2  |107 |6  |524|40 |400 |17 |26 |81 |118 |0  |13 |46 |33 |93 |59 |20 |0  |114|0   |183|76 |7  |19 |45 |90 |91 |
|t     |3  |145 |10 |270|331|441 |56 |52 |157|144 |6  |47 |149|39 |244|64 |40 |4  |152|344 |0  |42 |13 |5  |9  |29 |4  |
|u     |6  |461 |5  |140|33 |677 |17 |63 |89 |399 |1  |42 |103|30 |85 |488|17 |7  |243|152 |69 |0  |13 |157|2  |41 |1  |
|v     |1  |15  |18 |6  |20 |17  |83 |1  |8  |6   |1  |0  |14 |4  |11 |11 |0  |0  |19 |11  |28 |12 |0  |11 |0  |3  |0  |
|w     |1  |15  |2  |8  |12 |31  |4  |2  |19 |7   |0  |2  |28 |9  |24 |23 |0  |0  |42 |17  |9  |68 |5  |0  |0  |8  |0  |
|x     |0  |10  |0  |51 |1  |4   |1  |16 |1  |4   |0  |6  |0  |10 |3  |3  |3  |0  |9  |70  |6  |4  |3  |0  |0  |1  |14 |
|y     |2  |56  |3  |27 |34 |568 |2  |51 |38 |276 |0  |4  |44 |2  |43 |18 |6  |0  |87 |76  |54 |30 |2  |4  |2  |0  |3  |
|z     |0  |0   |0  |9  |5  |4   |0  |2  |1  |0   |0  |0  |1  |0  |1  |1  |0  |0  |1  |95  |1  |0  |1  |0  |2  |0  |0  |

- ### 1.3 Del Matrix

|Letter|" " |a  |b  |c  |d  |e  |f  |g  |h  |I  |j  |k  |l  |m  |n  |o  |p  |q  |r  |s  |t  |u  |v  |w  |x  |y  |z  |
|------|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|" "   |0  |133|40 |87 |21 |60 |26 |24 |57 |107|9  |53 |8  |9  |9  |35 |191|0  |32 |115|53 |16 |23 |61 |0  |5  |0  |
|a     |0  |0  |90 |374|71 |10 |92 |131|0  |264|0  |3  |652|75 |453|3  |267|0  |322|165|341|219|20 |3  |0  |27 |0  |
|b     |0  |18 |1  |0  |0  |142|0  |0  |0  |46 |0  |0  |56 |0  |0  |43 |0  |0  |39 |5  |11 |22 |1  |0  |0  |2  |0  |
|c     |0  |111|0  |33 |0  |301|0  |0  |341|383|0  |60 |18 |0  |0  |239|0  |1  |59 |32 |147|69 |0  |0  |0  |14 |0  |
|d     |0  |53 |0  |0  |0  |173|0  |32 |1  |191|4  |1  |17 |0  |1  |26 |0  |0  |15 |21 |1  |34 |5  |2  |0  |12 |0  |
|e     |0  |458|1  |304|665|14 |47 |45 |9  |95 |30 |0  |227|111|521|81 |27 |50 |499|433|143|47 |12 |8  |22 |30 |8  |
|f     |1  |60 |0  |1  |0  |59 |15 |0  |0  |142|0  |0  |26 |0  |0  |54 |0  |0  |99 |0  |15 |38 |0  |0  |0  |1  |0  |
|g     |0  |66 |0  |0  |0  |200|0  |15 |137|72 |0  |0  |13 |0  |39 |6  |0  |0  |70 |13 |5  |189|0  |0  |0  |7  |0  |
|h     |0  |57 |0  |0  |0  |309|0  |0  |0  |85 |0  |0  |17 |3  |14 |84 |0  |0  |7  |8  |41 |30 |0  |0  |0  |14 |0  |
|I     |0  |244|15 |289|62 |227|66 |207|0  |0  |0  |2  |132|131|478|233|67 |0  |53 |284|242|13 |48 |0  |0  |0  |12 |
|j     |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |17 |0  |0  |0  |0  |0  |29 |0  |0  |0  |0  |0  |
|k     |0  |0  |0  |0  |0  |106|0  |0  |0  |28 |0  |0  |0  |0  |16 |0  |0  |0  |0  |21 |0  |0  |0  |0  |0  |0  |0  |
|l     |0  |88 |0  |3  |14 |381|11 |1  |0  |220|0  |3  |72 |0  |0  |139|3  |0  |6  |24 |20 |5  |14 |0  |0  |144|0  |
|m     |0  |137|54 |0  |0  |237|17 |0  |0  |124|0  |0  |0  |16 |20 |78 |91 |0  |0  |18 |0  |23 |0  |0  |0  |2  |0  |
|n     |0  |297|0  |217|169|526|6  |198|0  |307|5  |3  |13 |2  |21 |76 |3  |11 |0  |282|367|27 |23 |0  |31 |23 |0  |
|o     |0  |67 |15 |55 |32 |40 |26 |41 |1  |42 |1  |3  |100|220|250|3  |148|0  |249|67 |80 |491|30 |115|17 |40 |1  |
|p     |0  |97 |0  |0  |0  |245|0  |0  |90 |52 |0  |0  |80 |1  |14 |56 |20 |0  |163|8  |60 |10 |0  |0  |0  |1  |0  |
|q     |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |78 |0  |0  |0  |0  |0  |
|r     |0  |322|1  |83 |26 |668|13 |18 |52 |383|0  |3  |6  |23 |83 |184|8  |0  |9  |76 |152|31 |3  |2  |0  |55 |0  |
|s     |0  |79 |0  |383|1  |246|10 |0  |128|264|0  |27 |6  |5  |1  |98 |67 |34 |0  |20 |261|133|0  |54 |0  |40 |0  |
|t     |0  |332|0  |9  |0  |939|1  |7  |111|563|0  |0  |49 |6  |3  |130|0  |0  |174|151|11 |133|0  |9  |0  |17 |0  |
|u     |0  |127|33 |93 |26 |63 |30 |83 |0  |143|0  |0  |124|31 |132|14 |41 |9  |206|96 |70 |0  |3  |0  |0  |0  |0  |
|v     |0  |52 |0  |0  |0  |196|0  |0  |0  |56 |0  |0  |0  |0  |0  |23 |0  |0  |1  |0  |0  |0  |0  |0  |0  |0  |0  |
|w     |0  |29 |0  |0  |3  |65 |0  |0  |118|15 |0  |0  |11 |0  |2  |21 |0  |0  |4  |2  |0  |0  |0  |0  |0  |1  |0  |
|x     |0  |17 |0  |37 |0  |8  |0  |0  |52 |36 |0  |0  |0  |0  |0  |0  |8  |4  |0  |0  |7  |0  |0  |0  |0  |1  |0  |
|y     |1  |12 |4  |19 |1  |44 |0  |1  |0  |9  |0  |0  |2  |24 |0  |8  |2  |0  |2  |40 |1  |0  |0  |1  |0  |0  |0  |
|z     |0  |2  |0  |0  |0  |9  |0  |0  |0  |6  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |

 - ### 1.4 Ins Matrix

|Letter|" "|a  |b  |c  |d  |e  |f  |g  |h  |I  |j  |k  |l  |m  |n  |o  |p  |q  |r  |s  |t  |u  |v  |w  |x  |y  |z  |
|------|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|" "   |0  |133|40 |87 |21 |60 |26 |24 |57 |107|9  |53 |8  |9  |9  |35 |191|0  |32 |115|53 |16 |23 |61 |0  |5  |0  |
|a     |0  |0  |90 |374|71 |10 |92 |131|0  |264|0  |3  |652|75 |453|3  |267|0  |322|165|341|219|20 |3  |0  |27 |0  |
|b     |0  |18 |1  |0  |0  |142|0  |0  |0  |46 |0  |0  |56 |0  |0  |43 |0  |0  |39 |5  |11 |22 |1  |0  |0  |2  |0  |
|c     |0  |111|0  |33 |0  |301|0  |0  |341|383|0  |60 |18 |0  |0  |239|0  |1  |59 |32 |147|69 |0  |0  |0  |14 |0  |
|d     |0  |53 |0  |0  |0  |173|0  |32 |1  |191|4  |1  |17 |0  |1  |26 |0  |0  |15 |21 |1  |34 |5  |2  |0  |12 |0  |
|e     |0  |458|1  |304|665|14 |47 |45 |9  |95 |30 |0  |227|111|521|81 |27 |50 |499|433|143|47 |12 |8  |22 |30 |8  |
|f     |1  |60 |0  |1  |0  |59 |15 |0  |0  |142|0  |0  |26 |0  |0  |54 |0  |0  |99 |0  |15 |38 |0  |0  |0  |1  |0  |
|g     |0  |66 |0  |0  |0  |200|0  |15 |137|72 |0  |0  |13 |0  |39 |6  |0  |0  |70 |13 |5  |189|0  |0  |0  |7  |0  |
|h     |0  |57 |0  |0  |0  |309|0  |0  |0  |85 |0  |0  |17 |3  |14 |84 |0  |0  |7  |8  |41 |30 |0  |0  |0  |14 |0  |
|I     |0  |244|15 |289|62 |227|66 |207|0  |0  |0  |2  |132|131|478|233|67 |0  |53 |284|242|13 |48 |0  |0  |0  |12 |
|j     |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |17 |0  |0  |0  |0  |0  |29 |0  |0  |0  |0  |0  |
|k     |0  |0  |0  |0  |0  |106|0  |0  |0  |28 |0  |0  |0  |0  |16 |0  |0  |0  |0  |21 |0  |0  |0  |0  |0  |0  |0  |
|l     |0  |88 |0  |3  |14 |381|11 |1  |0  |220|0  |3  |72 |0  |0  |139|3  |0  |6  |24 |20 |5  |14 |0  |0  |144|0  |
|m     |0  |137|54 |0  |0  |237|17 |0  |0  |124|0  |0  |0  |16 |20 |78 |91 |0  |0  |18 |0  |23 |0  |0  |0  |2  |0  |
|n     |0  |297|0  |217|169|526|6  |198|0  |307|5  |3  |13 |2  |21 |76 |3  |11 |0  |282|367|27 |23 |0  |31 |23 |0  |
|o     |0  |67 |15 |55 |32 |40 |26 |41 |1  |42 |1  |3  |100|220|250|3  |148|0  |249|67 |80 |491|30 |115|17 |40 |1  |
|p     |0  |97 |0  |0  |0  |245|0  |0  |90 |52 |0  |0  |80 |1  |14 |56 |20 |0  |163|8  |60 |10 |0  |0  |0  |1  |0  |
|q     |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |78 |0  |0  |0  |0  |0  |
|r     |0  |322|1  |83 |26 |668|13 |18 |52 |383|0  |3  |6  |23 |83 |184|8  |0  |9  |76 |152|31 |3  |2  |0  |55 |0  |
|s     |0  |79 |0  |383|1  |246|10 |0  |128|264|0  |27 |6  |5  |1  |98 |67 |34 |0  |20 |261|133|0  |54 |0  |40 |0  |
|t     |0  |332|0  |9  |0  |939|1  |7  |111|563|0  |0  |49 |6  |3  |130|0  |0  |174|151|11 |133|0  |9  |0  |17 |0  |
|u     |0  |127|33 |93 |26 |63 |30 |83 |0  |143|0  |0  |124|31 |132|14 |41 |9  |206|96 |70 |0  |3  |0  |0  |0  |0  |
|v     |0  |52 |0  |0  |0  |196|0  |0  |0  |56 |0  |0  |0  |0  |0  |23 |0  |0  |1  |0  |0  |0  |0  |0  |0  |0  |0  |
|w     |0  |29 |0  |0  |3  |65 |0  |0  |118|15 |0  |0  |11 |0  |2  |21 |0  |0  |4  |2  |0  |0  |0  |0  |0  |1  |0  |
|x     |0  |17 |0  |37 |0  |8  |0  |0  |52 |36 |0  |0  |0  |0  |0  |0  |8  |4  |0  |0  |7  |0  |0  |0  |0  |1  |0  |
|y     |1  |12 |4  |19 |1  |44 |0  |1  |0  |9  |0  |0  |2  |24 |0  |8  |2  |0  |2  |40 |1  |0  |0  |1  |0  |0  |0  |
|z     |0  |2  |0  |0  |0  |9  |0  |0  |0  |6  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |0  |


## 2. Screenshots  

![Screenshot from 2020-04-08 06-48-34](https://user-images.githubusercontent.com/32520017/78742626-3cfb9580-7965-11ea-91f6-5e9ca3fa2c3c.png)

![Screenshot from 2020-04-08 02-27-52](https://user-images.githubusercontent.com/32520017/78742369-7b448500-7964-11ea-9d7b-52622c8b7c77.png)

![Screenshot from 2020-04-08 02-27-52](https://user-images.githubusercontent.com/32520017/78742412-a0d18e80-7964-11ea-8b4a-747460832b45.png)

![Screenshot from 2020-04-08 01-08-46](https://user-images.githubusercontent.com/32520017/78742476-dc6c5880-7964-11ea-9c20-7309e1fd0369.png)

![Screenshot from 2020-04-08 06-51-46](https://user-images.githubusercontent.com/32520017/78742751-8ba92f80-7965-11ea-919d-4dfabc5d9fe3.png)

![Screenshot from 2020-04-08 01-12-28](https://user-images.githubusercontent.com/32520017/78743189-97492600-7966-11ea-86cb-da22cd4ad3d6.png)


## 3. Accuracy Scores

 - ### 3.1 Results without Smoothing

     Without smoothing Spelling Error Corrector has corrected 316 words over 384 misspelled words.
     Number of 252 corrected words over those 316 words are true according to the provided test file.
     That means accuracy is 79.74 

 - ### 3.2 Results with Smoothing
  
      Without smoothing Spelling Error Corrector has corrected 316 words over 384 misspelled words.
      Number of 252 corrected words over those 316 words are true according to the provided test file.
      Accuracy is again 79.74. In my personal opinion, small size of speell-errors.txt and test-words-misspelled.txt
      cause not to observe a difference between smoothing and without it.

 - ### 3.2 Results with Edit Distance Smaller than or equal to 2 (Advanced)

    In the **advanced** mode program select candidates that their edit distance smaller than or equal to 2.
    I had called this mode **advanced** because I had expected accuracy to increase. However It did not.
    In advanced mode, Spelling Error Corrector has corrected 354 words over 384 misspelled words. Increasing number of
    corrected words by 38 was observed. And, Number of 266 corrected words over those 354 words are true according to the provided test file.
    Increasing number of true conversion by 14. Accuracy in advanced mode is 75.14, clearly it shouldn't had been named **advanced**.
Nevertheless, it could be used in situations where accuracy is not as important as giving any correct solution.
![Screenshot from 2020-04-08 01-06-31](https://user-images.githubusercontent.com/32520017/78743154-797bc100-7966-11ea-80db-61ae8485ff32.png)

## 4. Errors and Possible Improvements

The Spell Error Corrector can not find a solution or finds incorrect words for some of misspelled words.
In my opinion, errors could be decreased by extending the reference corpus file, in other words this done by enriching the dictionary.
Second, errors is related with the confusion matrices. Confusion matrices are constructed according to error-misspelled.txt file.
By enriching error-misspelled.txt would decrease errors.  
