
so basically what i am trying to do here is get the shit together and jotting down some info about TensorRT

i use it for basically running my pre-trained model it doesn't have powers to train the model it just gives me the answers takes less time and also less gpu
<br>
(why am i using yolo(https://www.youtube.com/watch?v=etjkjZoG2F0) for training my model? and not raw TensorRT? (might be able to get better results but it will take a lot of time will get back at it in some later time))
<br>

since you have a pretrained model (or will train one), here is the standard industrial pipeline you need to follow tonight.

The Pipeline: PyTorch (Training) -> ONNX (The Bridge)(open neural network exchange) -> TensorRT (The Engine)

<br>

Concept: TensorRT is a compiler that makes AI models run faster after they are trained.

Usage: Used to save costs (servers) or reduce latency (autonomous cars).

Action:

    Take your trained .pt file.

    Run the Python script above to make an .onnx file.

    Run the trtexec command to make the .engine file.

<br>

just run the scripts on terminal and the engine file gets created 
