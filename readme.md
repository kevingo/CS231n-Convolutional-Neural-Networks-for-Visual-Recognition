本堂課首先介紹了什麼是一個 image classification 的問題。主要是給定一堆圖片和該圖片的類別，透過電腦來自動針對圖片進行分類。

以一張 280x400 的彩色圖片來說，電腦所看到的「影像」會是一個 280x400x3 的陣列，總共 297,600 個數字。

在進行影像分類時，你期待結果會是每個分類對應的機率，你挑一個機率最大的當成標準答案。

![img_classification](https://raw.githubusercontent.com/kevingo/blog/master/screenshot/img_classification.png)

而要訓練一個好的影像分類器其實是不容易的，因為影像有以下的一些特性，比如說不同角度、不同的 scale，相對位置的不同，這些變形都必須要能夠被正確的分類才行。

![img_diff](https://raw.githubusercontent.com/kevingo/blog/master/screenshot/img_clf_diff.png)

使用 Nearest Neighbor 分類器來分類影像
- 每張測試圖片和訓練圖片進行像素之間的比較，相減之後取絕對值「相素差」加總
- 對每一張訓練圖片做完上述計算後，取「相素差」最小的那一張圖片當做分類結果

![img_nn](https://raw.githubusercontent.com/kevingo/blog/master/screenshot/img_nn.png)

上面這種取距離的方法叫做「L1 distance」(也叫做 Manhattan distance)，

![l1_dis](https://raw.githubusercontent.com/kevingo/blog/master/screenshot/l1_dis.png)

另一種計算距離的方法叫做「L2 distance」(也就是歐幾里德距離)，則是相減平方後開根號：

![l2_dis](https://raw.githubusercontent.com/kevingo/blog/master/screenshot/l2_dis.png)

通常，我們會使用 KNN，也就是不只看最近的一個，不只看最近的一個訓練資料來決定測試資料的類別，而是會看 K 個訓練資料後，根據這 K 個樣本投票決定該測試資料最後的樣本。

一般來說，這樣比較不會受到 outlier 的影響。

![knn](https://raw.githubusercontent.com/kevingo/blog/master/screenshot/knn.png)

NN 分類法的優缺點
- 優點
    - 方法簡單易懂
    - 分類器不需要花時間訓練，只需要將訓練資料的 index 儲存起來即可
- 缺點
    - 驗證測試資料時很花時間，需要算過所有的訓練資料

一般來說，在如此高維度的影像中，很少使用 NN 當作分類器。課程中提供了一個範例，左邊是原始圖片，右邊三張圖片對於原始圖片來說，在以 L2 distance 的 KNN 演算法來說都會被歸為不同類別(距離很遠)：

![knn_img](https://raw.githubusercontent.com/kevingo/blog/master/screenshot/knn_img.png)