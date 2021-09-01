# FEN-Chess-Identifier-APS360-Summer-2021
APS360 Summer 2021 Team Repo (Group 3)
Team members: John Lee, Kevin Karam, Morgan Tran, Joanne Tan

# Introduction 
The goal of this project is to generate Forsyth-Edwards Notation (FEN) descriptions for a particular chess game from a picture of a 2D-board. Machine learning is a reasonable approach for this project as it involves multi-classification of 12 chess pieces and properly outputting FEN. The motivation behind the project was based on an overall team interest in chess and how to effectively store and use the chessboard data (FEN) from games. With the recent rise in popularity in chess, this project has big implications on the chess community where people can easily take images from a professional game and quickly analyse it. 

# Overview of Project Use Case 
![FEN Generation Process Flow Chart](https://user-images.githubusercontent.com/32041321/131747852-ec885f75-1816-4507-9f94-7ee0fea20d46.jpeg)

# Demo/Presentation
Link to full presentation [here](https://drive.google.com/file/d/1siRZj-PIeo-A3_XFDpK9VavtTFkdu6I8/view?usp=sharing).
For demo, please refer to timestamp 4:25 - 5:38

Quick reference:
1. Taking screenshot from a video, we input the image into our model
2. Our model outputs an image with bounding boxes and class predicitions of chess pieces it has detected
3. Reading the text file generated by our model (running some code), we extract the bounding box predictions and convert to FEN
4. Check our work: copying the generated FEN notation, paste it in lichess to recreate the chess board from the screenshot
5. Ta-da! The generated board on lichess is the same as the screenshot :) 
![Demo clip GIF](https://user-images.githubusercontent.com/32041321/131751559-9a77d8e5-f90b-4e72-831e-b20c344c190e.gif)


# Baseline Model 
To evaluate our model our team needed to find a method that would be better than randomly guessing. The team decided to make a hand coded heuristic model that just identifies the piece and colour when given a square image of a chess piece. We did this by first setting aside a base set of “default” pieces to compare against. This base set was then converted to numpy arrays so we could easily compare them against the input. When the baseline model gets an input image, it converts it to a numpy array and gets the HSP value of the middle pixel. The HSP value helps us determine the input’s perceived brightness. We set a threshold value of 0.3 and if the image was over that, we guessed it to be white. Under the threshold resulted in guessingblack. The input numpy arrays were then compared against the base images pixel-by-pixel. The amount of matching pixels were stored and the model outputted the piece that matched the most. Purely guessing would theoretically give you an accuracy of 8.33%. Our baseline model ended up having an accuracy of 14%.

# Architecture
The final model used for our results is YOLOv5 (a variation of ‘You Only Look Once’ object detection model, pre-trained on COCO dataset). This model can be accessed through Pytorch and at the GitHub repository made by Ultralytics [4]. YOLOv5 has 9 different variations, each variation differing in the number of parameters and expected image resolution. Due to the computation limit of Google Colab and image resolution of our dataset, we decided to utilize YOLOv5x which has ~87 million parameters [4]. The YOLOv5 model is made of a backbone (trained on CSPDarknet) that does feature extraction, a neck (PANet) that does feature fusion, and a head (YOLO layer) that does the detection (Figure 3). YOLOv5 uses various convolutional layers, max/average pooling, bottlenecks, SPP, activation functions (LeakyReLU, SiLU, Mish, Swish), concatenations, and upsamplings [5][6][7]. 

Using a custom dataset, we trained the YOLOv5 model to identify all instances of a chess piece in a given image on a chess board. By using a SGD optimizer, a batch size of 15, and epoch of 100 (saving the best model out of 100 epochs), the model was able to achieve a 90% accuracy on the test set. The confidence and IOU (intersection over union) thresholds were set to 0.5. Refer to Appendix A for Google Colab notebooks for model code.

<div align="center">
  <img src="https://user-images.githubusercontent.com/32041321/131752681-80ecd6be-8903-4f5f-9a8f-5a89fbc793c4.jpg" />
  <p>Figure: YOLOv5 architecture [7] </p>
</div>



# Why Object Detection?
An object detection model was used instead of a regular classification model because we needed a way to locate the exact position of the chess piece detected on the board. Through a regular convolution model, we would have to teach the model to identify the grids on the board, and the infinite combinations of chess positions. Thus, it seemed much easier to use an object detection model and use the bounding boxes predicted to determine the position of the piece on the board instead; with the assumption that the image given to the model is of only the chess board with pieces (2D) in their respective squares. 

# Training Results
Please refer to the logged training results at [Weights and Biases](https://wandb.ai/aps360-group3-2021/YOLOv5/reports/FEN-Generator-APS360-Summer-2021-Training-Report-Results--Vmlldzo5ODUzMjY?accessToken=wzxixfsd9pmub4cqhmlsfahkszqpaxerv1xeu3rdfhnbkoi0g5ionlw1zqnjko72)

# Results 
Overall the results of our model were surprisingly good. In our Precision plot, only the black rook had a low spike in precision, but generally, precision was at 1.00 precision at 0.845 confidence. 

![Confusion-matrix](https://user-images.githubusercontent.com/32041321/131752455-69c09527-06ec-4b25-81d2-9cc956948c84.jpg)
![F1-curve](https://user-images.githubusercontent.com/32041321/131752462-ff5f1a38-6460-419f-8d73-35aae59c33d5.jpg)
![Precision-curve](https://user-images.githubusercontent.com/32041321/131752482-9e6b25e0-dba9-4637-b909-aac052c1f2b4.jpg)
![Recall-curve](https://user-images.githubusercontent.com/32041321/131752484-90f3a8ab-890d-4b8a-a9e8-fd9e24c65fe1.jpg)
![PR-curve](https://user-images.githubusercontent.com/32041321/131752477-de2c8278-88de-464a-894a-8c9efb97bc9d.jpg)
![Training and metrics curves](https://user-images.githubusercontent.com/32041321/131748768-06eb18e6-f2e7-4489-b700-65ca7f38c3ca.png)




# References
[1] L. Spears, “Transcribe Live Chess with Machine Learning Part 1,” Medium, 25-Aug-2019. [Online]. Available: https://towardsdatascience.com/transcribe-live-chess-with-machine-learning-part-1-928f73306e1f. [Accessed: 03-Jun-2021].

[2] D. M. Quintana, A. A. del B. Garc´ıa, and M. P. Mat´ıas, “LiveChess2FEN: a Framework for Classifying Chess Pieces based on CNNs,” arxiv, 15-Dec-2020. [Online]. Available: https://arxiv.org/pdf/2012.06858.pdf. [Accessed: 02-Jun-2021].

[3]  P. Koryakin, “Chess Positions,” Kaggle, 03-Feb-2019. [Online]. Available: https://www.kaggle.com/koryakinp/chess-positions. [Accessed: 01-Jun-2021].
Recognise chess position of 5-15 pieces, Version 1

[4] Ultralytics. (2021) YOLOv5 (Version 5) [Source code]. https://github.com/ultralytics/yolov5. [Accessed: 01-Jul-2021].

[5] Ultralytics. (2021) YOLOv5 (Version 5) [Source code]. https://github.com/ultralytics/yolov5/blob/master/models/yolov5x.yaml. [Accessed: 01-Jul-2021].

[6]  Ultralytics. (2021) YOLOv5 (Version 5) [Source code]. https://github.com/ultralytics/yolov5/blob/master/models/common.py. [Accessed: 01-Jul-2021].

[7] R. Xu, H. Lin, K. Lu, L. Cao, and Y. Liu, “A forest fire detection system based on ensemble learning,” Forests, vol. 12, no. 2, p. 217, 2021.
Refer to figure of YOLOv5 network architecture
