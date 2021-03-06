# 影像分類

**動機。** 在本節中，我們會介紹影像分類的問題，這類問題的目的是從一個固定集合的類別中，指派一個類別到給訂的影像上。這在電腦視覺的領域上，是一個核心的問題。儘管它看起來很簡單，但在實務上有許多的應用。此外，我們會在接下來的課程中看到，有許多的問題 (像是物件偵測、物件分割) 都可以被收斂為影像分類的問題。

**範例。** 舉例來說，底下的影像分類模型會接收一個影像的輸入，並給出此張影片屬於 4 個類別 {貓、狗、帽子、杯子} 分別的機率。要注意的是，對於電腦來說，一張影像會被表示為三維陣列的數字。在本例中，這張貓的圖片寬 248 像素、高 400 像素，並且為三個通道 - 紅、綠、藍，的圖片 (或是我們可以用 RGB 表示)。因此，這張圖片包含 248 x 400 x 3 個數字，總共 297,600 個數字。每一個數字的範圍從 0 (黑色) 到 255 (白色)。我們的任務是將這一堆數字給訂一個類別，比如說 "貓"。

---
![cat](https://raw.githubusercontent.com/kevingo/CS231n-Convolutional-Neural-Networks-for-Visual-Recognition/master/images/classify.png)

影像辨識的任務是給定一張圖片，要去預測圖片是屬於哪個類別 (或是給出該張圖片屬於各個類別的機率值)。影像三維的整數陣列，值會從 0 到 255，陣列的大小是長*寬*3。3 代表的是三個顏色通道，紅、綠、藍。

---

**挑戰** 由於辨識一個圖像對於人類來說是相對簡單的，但讓電腦來進行這樣的挑戰就相對困難了。相對困難了。底下我們列舉了一些影像識別會遇到的困難，要記住圖片是三維的，其中的元素代表的是亮度：

- 視角變化。照相機可以從多個角度來拍攝一個物體。
- 大小變化。物體的大小是會變化的 (不只在圖片中會變化，真實世界也可能改變大小)。
- 形變。很多物體的形狀並非一成不變，有可能會有很大的變化。
- 遮蔽。目標的物體可能被遮蔽。有時候只有目標物體的一小部分 (可以小到幾個像素) 是可見的。
- 光照條件。像素層面的光照影響非常大。
- 背景干擾。目標物體很可能會混入背景中，變得相當難以識別。
- 類別內差異。目標物體可能存在很在的差異，舉例來說，世界上的椅子可能有千奇百怪的種類，每一種椅子都有自己的型態。

一個好的影像分類模型要能夠克服以上的困難，並且能夠區份不同類別之間的差異。

---
![image](https://raw.githubusercontent.com/kevingo/CS231n-Convolutional-Neural-Networks-for-Visual-Recognition/master/images/challenges.jpeg)

---

**以資料驅動的方法** 如何撰寫一個影像分類的演算法呢？這可能跟我們在寫排序演算法有大大的不同。要寫一個辨識貓的圖片的演算法要怎麼寫？因此，與其在程式碼中寫明要怎麼辨識每個類別的圖片，我們採取的作法跟教幼童辨識物體的方法類似：我們針對每個類別，給電腦看許多該類別的圖片，讓電腦學習如何辨識每個圖片。這種方法叫做資料驅動的方法。這種方法首先必須要搜集許多的圖片作為訓練資料集，底下讓我們來看看訓練資料集會長怎樣：

---
![image](https://raw.githubusercontent.com/kevingo/CS231n-Convolutional-Neural-Networks-for-Visual-Recognition/master/images/trainset.jpg)

上圖是四種類別的範例圖片。在實務上，我們可能會有數千個類別，每個類別會有數百或數千張圖片。

---

**影像分類流程** 我們已經看到，圖片分類的任務就是將圖片透過陣列的方式當成輸入，透過分類模型給訂一個類別。完整的流程如下：

- **輸入：** 我們的輸入包含 N 張圖片，每張圖片是 K 個類別中的其中一個，這些資料稱為訓練資料集。
- **學習：** 這一步驟的任務是要學習每個類別的圖片長得如何。一般來說，這個步棸稱為訓練分類器，或學習一個分類模型。
- **評估：** 最後，我們透過讓分類器來預測它沒看過的圖片的類別，以此來評估此分類器的效能。我們把分類器預測的類別和真實類別進行比較。直觀上來說，我們希望分類器預測的類別和真實類別一致。

## 最近鄰居分類器

作為這門課程的第一個方法，我們來開發一個方法，叫做最近鄰居分類器。這個分類器跟卷積神經網路沒有任何關係，在實務上也極少使用，但學習它對於我們對於解決圖片分類問題會有一個基本的認識。

**圖片分類資料集： CIFAR-10.**
一個非常流行的圖片分類資料集叫做 CIFAR-10。這個資料集包含 60,000 張小圖，每張圖片屬於 10 個類別其中之一 (比如說飛機、汽車、鳥等等)。這 60,000 張圖片被分為 50,000 張訓練集，以及 10,000 張測試集。底下你可以看到 10 個類別與其對應隨機挑選出來的圖片：

---
![image](https://raw.githubusercontent.com/kevingo/CS231n-Convolutional-Neural-Networks-for-Visual-Recognition/master/images/nn.jpg)

- 上圖左：CIFAT-10 的範例圖片。
- 上圖右：第一列是測試圖片，而右邊是每個測試圖片根據最近鄰居分類器找出來像素距離最接近的 10 張圖片。

---

假設我們現在已經有 50,000 張 CIFAR-10 的訓練圖片資料集 (每個類別 5000 張)，我們想要預測剩下的 10,000 張圖片的類別。最近鄰居分類器會針對每一張測試圖片，去尋找跟它距離最接近的訓練圖片來給出類別。上圖的右方你可以看到這種分類器的效果。請注意上方的結果，在十個類別中只有三個類別的分類結果是正確的，另外七個是比較差的。以第八列的馬頭為例，跟它最接近的是一輛紅色的車子，原因是該張車子的圖片的背景也是全黑的，所以演算法會誤認為這是跟測試圖片很接近的訓練資料。

你可能會注意到我們還沒有提到具體上我們怎麼比較這兩張 32 x 32 x 3 的圖片。一個最簡單的方法就是針對每個像素進行比較，換句話說，就是把兩張圖片用向量表示為 I1,I2，接著比較兩者的 **L1 距離：**

這裡的總和指的是所有像素總和。底下是相關的圖例：

---
![image](https://raw.githubusercontent.com/kevingo/CS231n-Convolutional-Neural-Networks-for-Visual-Recognition/master/images/nneg.jpeg)

上圖是使用 L1 距離來比較兩張圖片像素距離的範例 (單一顏色通道)。所有的差值相加後得到一個數值，如果兩張圖片相同，L1 距離應該是零。但如果兩張圖片相差很大，L1 距離差應該很大。

---

讓我們來看看怎麼實作這個分類器。首先，讓我們讀取 CIFAR-10 的資料到記憶體中，並把他們儲存成四份陣列資料：訓練集/訓練的類別標籤、測試集/測試的類別標籤。底下程式碼的 `Xtr` (大小為 50,000 x 32 x 32 x 3) 儲存所有訓練資料集的圖片，而一維陣列 `Ytr` (長度為 50,000) 則儲存訓練資料級的類別標籤 (從 0 到 9)：

```python
Xtr, Ytr, Xte, Yte = load_CIFAR10('data/cifar10/') # a magic function we provide
# flatten out all images to be one-dimensional
Xtr_rows = Xtr.reshape(Xtr.shape[0], 32 * 32 * 3) # Xtr_rows becomes 50000 x 3072
Xte_rows = Xte.reshape(Xte.shape[0], 32 * 32 * 3) # Xte_rows becomes 10000 x 3072
```

現在我們擁有一個所有圖片所形成的向量，底下我們就來訓練和評估分類器：

```python
nn = NearestNeighbor() # create a Nearest Neighbor classifier class
nn.train(Xtr_rows, Ytr) # train the classifier on the training images and labels
Yte_predict = nn.predict(Xte_rows) # predict labels on the test images
# and now print the classification accuracy, which is the average number
# of examples that are correctly predicted (i.e. label matches)
print 'accuracy: %f' % ( np.mean(Yte_predict == Yte) )
```

在評估分類性能的指標上，我們經常會使用 **準確率(accuracy)** 作為衡量的基準，它代表了我們預測正確的比例有多少。要特別注意的是，往後我們建立的所有分類器都會有這個 API：`train(X,y)` 函式，用來讀取資料並且進行訓練。從內部來看，這個類別負責從類別標籤和資料訓練出一個模型。另外有一個 `predict(X)` 函式，用來針對新看到的資料進行預測。沒錯，我們省略了最重要的分類器的實作部分，底下是一個使用 L1 距離函式所建立的最近鄰居分類器：

```python
import numpy as np

class NearestNeighbor(object):
  def __init__(self):
    pass

  def train(self, X, y):
    """ X is N x D where each row is an example. Y is 1-dimension of size N """
    # the nearest neighbor classifier simply remembers all the training data
    self.Xtr = X
    self.ytr = y

  def predict(self, X):
    """ X is N x D where each row is an example we wish to predict label for """
    num_test = X.shape[0]
    # lets make sure that the output type matches the input type
    Ypred = np.zeros(num_test, dtype = self.ytr.dtype)

    # loop over all test rows
    for i in xrange(num_test):
      # find the nearest training image to the i'th test image
      # using the L1 distance (sum of absolute value differences)
      distances = np.sum(np.abs(self.Xtr - X[i,:]), axis = 1)
      min_index = np.argmin(distances) # get the index with smallest distance
      Ypred[i] = self.ytr[min_index] # predict the label of the nearest example

    return Ypred
```

如果你執行以上的程式碼，你會看到該分類器在 CIFAR-10 上的正確率只有 38.6%。這樣的結果比起隨機猜測好得多 (大概是 10%)，但跟人類的表現 ([大約 94%](http://karpathy.github.io/2011/04/27/manually-classifying-cifar10/)) 和目前接近最好的卷積神經網路的 95% 比起來差多了 (你可以從 Kaggle 的[排行榜](http://www.kaggle.com/c/cifar-10/leaderboard)上查看目前的準確率排名)。

**距離函數的選擇** 向量之間距離的計算有許多方法。另一個常見的做法是使用 **L2 距離** 函式，從幾何學的角度來看，他是計算兩個向量之間的歐式距離，定義如下：

![image](https://raw.githubusercontent.com/kevingo/CS231n-Convolutional-Neural-Networks-for-Visual-Recognition/master/images/distance_func.png)

換句話說，我們依然是在比較像素之間的差異，但在這裡我們使用了平方相加後開根號的方式。在 Numpy 裡面，只要換掉上面的一行程式碼就可以了：

```python
distances = np.sqrt(np.sum(np.square(self.Xtr - X[i,:]), axis = 1))
```

在這裡我們使用了 `np.sqrt` 函式，在實務上可能不需要，因為平方根是一個單調函數，也就是說，雖然使用平方根改變的數值大小，但彼此之間的順序關係依舊維持，所以對最近鄰居分類器來說，有沒有使用平方根函數並不影響。如果你使用這個距離函式在 CIFAR-10 上，會得到 **35.4%** 的準確率 (比 L1 距離函式來得低一點)。

**L1 vs. L2.** 比較這兩種距離函式的差別是挺有趣的。特別是， L2 距離函式比起 L1 距離函式更不能接受兩個向量之間的差異。也就是說，L2 距離更容易接受許多介於中間的差異，而不是一個大的差異。這兩種距離函式(或這兩種距離函式在圖片上的差異) 都是經常用在 [p-norm](http://planetmath.org/vectorpnorm) 上的特別形式。

## k 個最近鄰居分類器

你可能會注意到我們僅僅使用距離最接近的圖片來進行預測有點奇怪。事實上，當你使用 k 個最近鄰居分類器時的做法就是如此。背後的概念非常簡單：與其使用一張最接近的圖片來做預測，我們使用 k 張圖片，最後投票來決定要預測哪個類別。當 k = 1 時，就是最近鄰居分類器了。直覺上來看，k 越大，可以使得分類的效果越平滑，對於異常值的抵抗能力越強：

---

![image](https://raw.githubusercontent.com/kevingo/CS231n-Convolutional-Neural-Networks-for-Visual-Recognition/master/images/knn.jpeg)

上圖顯示了最近鄰居分類器和 5 個最近鄰居分類器的效果，我們使用二維來表示，分成三類 (紅、藍、綠)。顏色的分界代表的是分類器的決策邊界，這裡使用的是 L2 距離函式。白色的部分是模糊的分類區域 (代表某張圖片距離兩個以上的類別都很接近)。值得注意的是，在最近鄰居分類器中，有問題的資料點 (例如：在藍色分類區域中的綠色點) 會建立出一個孤島，就像分類錯誤一樣，但在 5 個最近鄰居分類器上，這樣的不規則的分類邊界變得平滑了，這代表了分類器在測試資料上泛化的能力越好 (上圖例子中沒有顯示)。注意，在 5 個最近鄰居分類器的圖中，也有一些灰色的區域，這是因為鄰居區域的分類標籤票數相同導致 (例如：有兩個是紅色、兩個是藍色、一個是綠色)。

---

實務上，你會經常使用 k 個最近鄰居分類器。但我們怎麼決定 k 是多少呢？接下來我們就來討論這個問題。

## 使用驗證資料集來進行超參數的調優

k 個最近鄰居分類器需要決定 k 值。但什麼數字是最好的？此外，我們看到有許多的距離函式：L1、L2，另外還有許多我們沒有提到的 (例如：內積)。這幾個部分我們稱之為**超參數**，你會經常在設計機器學習演算法時看到他們。一般來說，如何選擇這些超參數並不是很直覺的。

你可以會嘗試各種不同的設定，看看哪一個表現最好。這是很好的想法，而且我們就是會這樣做。但要特別小心，我們在調整這些超參數時，不能夠使用測試資料集。當你在設計機器學習演算法時，你應該要把測試資料集當成是寶貴的資源，不到最後階段不會使用它們。否則，一個顯而易見的危險是，你會發現在測試資料集上的表現很好，但在部署上線時，效能會遠低於預期。實務上，這種現象稱之為對於測試資料集的過擬和 (overfitting)。另一方面來看，當你使用測試資料集來調整超參數時，你其實就是把測試資料集當成訓練資料集來使用，所以你測試出來的效能其實過於樂觀，會遠比在上線的時候要高出許多。但如果你在最後才使用測試資料集，這樣就可以很好的模擬訓練出來分類器的泛化程度 (我們會在後面討論更多關於泛化的內容)。

```
測試資料集只使用一次，就是在最後階段的時候。
```

幸運的是，我們有正確的方法來進行超參數的調整，並且不需要碰到測試資料集。概念是將你的訓練資料集分成兩部分：一個比較小的訓練資料集，或是我們稱之為驗證資料集。用 CIFAR-10 當範例的話，我們可以使用 49,000 張圖片當成訓練資料集，1,000 張圖片當成驗證資料集。驗證資料集本質上就是一個假的測試資料，用來調整超參數。

實際上我們可以這樣做：

```python
# assume we have Xtr_rows, Ytr, Xte_rows, Yte as before
# recall Xtr_rows is 50,000 x 3072 matrix
Xval_rows = Xtr_rows[:1000, :] # take first 1000 for validation
Yval = Ytr[:1000]
Xtr_rows = Xtr_rows[1000:, :] # keep last 49,000 for train
Ytr = Ytr[1000:]

# find hyperparameters that work best on the validation set
validation_accuracies = []
for k in [1, 3, 5, 10, 20, 50, 100]:

  # use a particular value of k and evaluation on validation data
  nn = NearestNeighbor()
  nn.train(Xtr_rows, Ytr)
  # here we assume a modified NearestNeighbor class that can take a k as input
  Yval_predict = nn.predict(Xval_rows, k = k)
  acc = np.mean(Yval_predict == Yval)
  print 'accuracy: %f' % (acc,)

  # keep track of what works on the validation set
  validation_accuracies.append((k, acc))
```

在最後的流程中，我們可以畫出一張圖表來顯示哪個 k 值的表現最好。這樣我們就用這個 k 值來評估最終的測試資料集的表現。

```
將你的資料切分成訓練資料集和驗證資料集。使用驗證資料集來進行所有的超參數調整。最後再使用測試資料集來驗證性能。
```

**交叉驗證** 有時候我們的訓練資料集可能很小 (因此，驗證資料集也很小)，這時候，有些人會使用一個更複雜的方法來進行超參數調整，那就是交叉驗證 (cross-validation)。以前述為例，我們就不是取前 1,000 張圖片當成驗證資料集，而是透過迭代的方式來獲得更好的 k 值設定。舉例來說，5-fold 交叉驗證指的就是我們會把訓練資料分割成五個等分，使用四份來訓練，剩下的一份作為驗證。我們可以迭代的來取出驗證資料集，並且評估效能，最後再取平均來得到最後的性能指標。

---
![image](https://raw.githubusercontent.com/kevingo/CS231n-Convolutional-Neural-Networks-for-Visual-Recognition/master/images/cvplot.png)

這是使用 5-fold 交叉驗證來對 k 值調整的關係。對於每一個 k 值，我們使用 4-folds 來訓練，用第五個 fold 來進行評估。因此，對於每個 k，我們會得到 5 個準確率的結果 (y 軸是準確率，每個值如上圖的點)。上圖的線是取平均後所畫出來的，上下邊界代表的是標準差。從上圖中可以看到，在特定的資料集上，最好的結果是 k=7 的情況 (對應上圖峰值的地方)。如果我們使用超過 5-folds，可以預期得到更平滑 (較少雜訊) 的曲線。

---

**實務應用** 實務上，人們並不是很喜歡使用交叉驗證，主要的原因在於計算成本很高。一般而言我們會將資料按照 50%-90% 的比例切分成訓練資料集和驗證資料集。然而這也是根據具體的情況而定，舉例來說，如果需要調整的超參數很多，你可能會想要使用比較大的驗證資料集，但如果驗證資料集的數量很小 (可能只有幾百筆資料)，那還是使用交叉驗證比較安全。一般來說，我們經常使用 3-fold、5-fold 或 10-fold 的交叉驗證。

---
![image](https://raw.githubusercontent.com/kevingo/CS231n-Convolutional-Neural-Networks-for-Visual-Recognition/master/images/crossval.jpeg)

常見的資料分割模式，會分割成訓練和測試資料集。訓練資料集會分成數個部分 (例如這裡分成五等份)，1 ~ 4 份會當成訓練資料，第五份資料當成驗證資料，用來調整超參數。交叉驗證時，則是分批把 1 ~ 5 份的其中一份當成驗證資料。而當最後我們找到了最佳的超參數時，就會拿測試資料來進行測試 (測試資料只使用一次)。

---

## 最近鄰居分類器的優點和缺點

來談談最近鄰居分類器的優點和缺點是值得的。一個明顯的優點是這種分類器很容易實作和瞭解。此外，分類器不需要時間來訓練，因為其訓練過程只需要將訓練資料儲存起來即可。然而，我們會花費計算成本在測試階段，因為在測試階段我們需要將每一個測試資料和訓練資料進行比對。這很明顯是一個缺點，因為實務上我們關注更關注測試更關注測試的效率。事實上，深度學習網路在這樣的權衡上選擇另外一個極端：它在訓練時需要花費非常多的成本，但一旦訓練完畢，要測試新的資料就會變得很容易。這樣的狀況非常適合在實務上使用。

另外，最近鄰居分類器的計算複雜度是一個熱門的研究領域，有一些**近似最佳鄰居 (ANN) 演算法**和函式庫可以用來幫助最近鄰居在資料集中查找的效率 (例如：FLANN)。這些演算法可以在準確率與空間/時間複雜度上進行取捨，而這些方法通常依賴於一個前處理/索引的步驟來建立 kdtree，或是執行 k-means 演算法。

最近鄰居分類器有時可能是一個好的選擇 (特別是在資料是低維度時)，但實務上應用在影像辨識的機會不高。因為圖片都是高維度的資料 (圖片包含許多像素)，而高維度上的距離通常跟直覺都是相反的。下圖展示了使用 L2 距離和人類感覺上的距離會有很大的不同：

---
![image](https://raw.githubusercontent.com/kevingo/CS231n-Convolutional-Neural-Networks-for-Visual-Recognition/master/images/samenorm.png)

以像素距離為基礎的計算方式在高維度資料 (特別是圖片) 是非常反直覺的。原始圖片 (上圖左) 和其他另外三張圖片的 L2 距離都一樣。很顯然的，以像素為基礎的距離差距並不能反映真實意義上的差距。
---

底下是另外一個案例用來告訴你，使用像素距離來比較影像是不適當的。我們可以使用一個視覺化的技巧，叫做 [t-SNE](http://homepage.tudelft.nl/19j49/t-SNE.html) 來將 CIFAR-10 的資料映射到二維空間上，並且保留距離的資訊。在這張圖片中，相鄰的圖片代表其 L2 距離較接近：

---
![image](https://raw.githubusercontent.com/kevingo/CS231n-Convolutional-Neural-Networks-for-Visual-Recognition/master/images/pixels_embed_cifar10.jpg)

上圖是使用 t-SNE 來將 CIFAR-10 圖片壓縮到二維空間的示意圖。相鄰的圖片代表在 L2 距離上是比較接近的。值得注意的是，相近的圖片會受到背景的影響，而不一定是他真實的所屬類別。[點此](http://cs231n.github.io/assets/pixels_embed_cifar10_big.jpg)來看比較高解析度的版本。

---

具體來看，相近的圖片更接近於顏色分佈接近的圖片，或是基於背景相近，而不是意義上的接近。舉例來說，一隻狗可能會跟一張青蛙的圖片相近，只因為他們的背景都是白色的。從理想上來看，我們必定是希望相同類別的圖片可以被分類在一起，且不會被其他的因素所影響 (像是背景)。要達到這樣的目的，我們必須繼續往前，不能停止在比較像素距離的階段。

## 總結

總結我們學習的成果：

- 我們介紹了影像分類的問題，即是給訂一個影像資料集與對應標籤，希望演算法能夠預測沒有標籤的測試影像，並且根據給訂的標籤結果評估準確率。
- 我們介紹了簡單的分類器 - 最近鄰居分類器。同時介紹了幾個超參數 (像是 k、不同的距離函數以及如何比較他們)，而要選取一個好的參數並不是這麼容易。
- 我們學習了如何正確的設定超參數：將你的資料集分成訓練集和驗證集，嘗試不同的超參數組合，並且選擇在驗證集表現最好的那組參數。
- 如果訓練資料集數量不足，可以考慮使用交叉驗證的方式，它能幫助我們在選取超參數時降低雜訊。
- 當我們找到最佳超參數時，實際使用這組參數在測試資料集上。
- 我們看到最近鄰居分類器在 CIFAR-10 上的準確率大概 40%，他在實作上很簡單，但需要儲存所有的訓練資料，並且在進行測試時的成本很高。
- 最後，我們討論了 L1 和 L2 距離用在評估以像素為主的圖片上的表現是不適合的，因為距離很容易因為背景的因素而忽略了圖片實際的意義。

在接下來的課程中，我們將會著手解決這些挑戰，最後達到 90% 的準確率。在模型訓練完畢後，我們就可以丟棄訓練資料集，並且它讓我們能夠在一毫秒的時間內來評估測試圖片。

## 總結：k 個最近鄰居分類器在實務上的應用

如果你想要在實務上應用 k 個最近鄰居分類器 (希望不是用在圖片上，或是你只打算建立一個基準時)，底下是我們建議的流程：

1. 處理資料：對圖片的特徵進行正規化 (例如：一張圖片一個像素)，讓圖片具有平均值為零以及單位方差的特性。我們會在後面的章節中更仔細的討論，本章節中不會進行討論，因為這裡的圖片的一致性較高，不會出現分布差異較大的情形，也因此不需要進行影像正規化的處理。
2. If your data is very high-dimensional, consider using a dimensionality reduction technique such as PCA (wiki ref, CS229ref, blog ref) or even Random Projections. 如果你的資料維度非常高，試著應用維度縮減的技巧，例如 PCA ([wiki 連結](http://en.wikipedia.org/wiki/Principal_component_analysis)、[CS229](http://cs229.stanford.edu/notes/cs229-notes10.pdf)、[部落格](http://www.bigdataexaminer.com/understanding-dimensionality-reduction-principal-component-analysis-and-singular-value-decomposition/)) 或[隨機投影](http://scikit-learn.org/stable/modules/random_projection.html)。
3. 將資料隨機分成訓練/驗證資料集，按照一般規則，你可以把 70%-90% 的資料當成訓練資料集，這取決於你有多少的超參數要調整，以及這些超參數對於結果的影響程度而定。如果需要預測的超參數很多，那就應該使用較大的驗證集來有效的進行預估。如果對於驗證資料集的數量有所擔憂，那就使用交叉驗證，只要你的運算資源足夠，使用交叉驗證總是很好的 (切割的份數越多，效果越好，但也越消耗資源)。
4. 在驗證資料集上進行性能調整，嘗試足夠多的 k 值以及不同的距離函式 (L1 和 L2 距離)。
5. 如果 k 個最近鄰居分類器所耗費的時間太久，考慮使用近似最佳鄰居分類演算法 (例如：[FLANN](http://www.cs.ubc.ca/research/flann/))
6. 將表現最好的超參數記錄下來，而有個問題是，要不要把最好的超參數放回訓練資料集再訓練一次呢？答案是：請不要這樣做。因為當我們把驗證資料集放回訓練資料集後，最優參數的組合可能又會改變了。在實務上，請不要這麼做，你要做的是直接使用測試資料集來測試產生最佳超參數的模型表現，得到測試資料集的準確率後，把這個當成你的 k 個最近鄰居分類器的實際性能。

## 延伸閱讀

底下是一些額外的連結提供參考：

- [A Few Useful Things to Know about Machine Learning](http://homes.cs.washington.edu/~pedrod/papers/cacm12.pdf), 內容中的第六章和本次內容相關，但整份文章都值得閱讀

- [Recognizing and Learning Object Categories](http://people.csail.mit.edu/torralba/shortCourseRLOC/index.html), ICCV 2005 上關於物件分類的內容

