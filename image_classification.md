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

As our first approach, we will develop what we call a Nearest Neighbor Classifier. This classifier has nothing to do with Convolutional Neural Networks and it is very rarely used in practice, but it will allow us to get an idea about the basic approach to an image classification problem.

Example image classification dataset: CIFAR-10. One popular toy image classification dataset is the CIFAR-10 dataset. This dataset consists of 60,000 tiny images that are 32 pixels high and wide. Each image is labeled with one of 10 classes (for example “airplane, automobile, bird, etc”). These 60,000 images are partitioned into a training set of 50,000 images and a test set of 10,000 images. In the image below you can see 10 random example images from each one of the 10 classes:

