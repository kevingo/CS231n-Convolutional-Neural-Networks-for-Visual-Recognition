# 影像分類

**動機。** 在本節中，我們會介紹影像分類的問題，這類問題的目的是從一個固定集合的類別中，指派一個類別到給訂的影像上。這在電腦視覺的領域上，是一個核心的問題。儘管它看起來很簡單，但在實務上有許多的應用。此外，我們會在接下來的課程中看到，有許多的問題 (像是物件偵測、物件分割) 都可以被收斂為影像分類的問題。

**範例。** 舉例來說，底下的影像分類模型會接收一個影像的輸入，並給出此張影片屬於 4 個類別 {貓、狗、帽子、杯子} 分別的機率。要注意的是，對於電腦來說，一張影像會被表示為三維陣列的數字。在本例中，這張貓的圖片寬 248 像素、高 400 像素，並且為三個通道 - 紅、綠、藍，的圖片 (或是我們可以用 RGB 表示)。因此，這張圖片包含 248 x 400 x 3 個數字，總共 297,600 個數字。每一個數字的範圍從 0 (黑色) 到 255 (白色)。我們的任務是將這一堆數字給訂一個類別，比如說 "貓"。

**挑戰** 由於辨識一個圖像對於人類來說是相對簡單的，但讓電腦來進行這樣的挑戰就相對困難了。相對困難了。底下我們列舉了一些影像識別會遇到的困難，要記住圖片是三維的，其中的元素代表的是亮度：

- 視角變化。照相機可以從多個角度來拍攝一個物體。
- 大小變化。物體的大小是會變化的 (不只在圖片中會變化，真實世界也可能改變大小)。
- 形變。很多物體的形狀並非一成不變，有可能會有很大的變化。
- 遮蔽。目標的物體可能被遮蔽。有時候只有目標物體的一小部分 (可以小到幾個像素) 是可見的。
- 光照條件。像素層面的光照影響非常大。
- 背景干擾。目標物體很可能會混入背景中，變得相當難以識別。
- 類別內差異。目標物體可能存在很在的差異，舉例來說，世界上的椅子可能有千奇百怪的種類，每一種椅子都有自己的型態。

一個好的影像分類模型要能夠克服以上的困難，並且能夠區份不同類別之間的差異。

**以資料驅動的方法** 如何撰寫一個影像分類的演算法呢？這可能跟我們在寫排序演算法有大大的不同。要寫一個辨識貓的圖片的演算法要怎麼寫？因此，與其在程式碼中寫明要怎麼辨識每個類別的圖片，我們採取的作法跟教幼童辨識物體的方法類似：我們針對每個類別，給電腦看許多該類別的圖片，讓電腦學習如何辨識每個圖片。這種方法叫做資料驅動的方法。這種方法首先必須要搜集許多的圖片作為訓練資料集，底下讓我們來看看訓練資料集會長怎樣：

**影像分類流程** 我們已經看到，圖片分類的任務就是將圖片透過陣列的方式當成輸入，透過分類模型給訂一個類別。完整的流程如下：

- **輸入：** 我們的輸入包含 N 張圖片，每張圖片是 K 個類別中的其中一個，這些資料稱為訓練資料集。
- **學習：** 這一步驟的任務是要學習每個類別的圖片長得如何。一般來說，這個步棸稱為訓練分類器，或學習一個分類模型。
- **評估：** 最後，我們透過讓分類器來預測它沒看過的圖片的類別，以此來評估此分類器的效能。我們把分類器預測的類別和真實類別進行比較。直觀上來說，我們希望分類器預測的類別和真實類別一致。

## 最近鄰居分類器

作為這門課程的第一個方法，我們來開發一個方法，叫做最近鄰居分類器。這個分類器跟卷積神經網路沒有任何關係，在實務上也極少使用，但學習它對於我們對於解決圖片分類問題會有一個基本的認識。

**圖片分類資料集： CIFAR-10.**
一個非常流行的圖片分類資料集叫做 CIFAR-10。這個資料集包含 60,000 張小圖，每張圖片屬於 10 個類別其中之一 (比如說飛機、汽車、鳥等等)。這 60,000 張圖片被分為 50,000 張訓練集，以及 10,000 張測試集。底下你可以看到 10 個類別與其對應隨機挑選出來的圖片：

假設我們現在已經有 50,000 張 CIFAR-10 的訓練圖片資料集 (每個類別 5000 張)，我們想要預測剩下的 10,000 張圖片的類別。最近鄰居分類器會針對每一張測試圖片，去尋找跟它距離最接近的訓練圖片來給出類別。上圖的右方你可以看到這種分類器的效果。請注意上方的結果，在十個類別中只有三個類別的分類結果是正確的，另外七個是比較差的。以第八列的馬頭為例，跟它最接近的是一輛紅色的車子，原因是該張車子的圖片的背景也是全黑的，所以演算法會誤認為這是跟測試圖片很接近的訓練資料。

你可能會注意到我們還沒有提到具體上我們怎麼比較這兩張 32 x 32 x 3 的圖片。一個最簡單的方法就是針對每個像素進行比較，換句話說，就是把兩張圖片用向量表示為 I1,I2，接著比較兩者的 **L1 距離：**

這裡的總和指的是所有像素總和。底下是相關的圖例：

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