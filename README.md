<h1 align="center"><img src="https://github.com/user-attachments/assets/2ae20423-af53-4835-b8d1-272a44db5823">
<br>Stockfish NNUE</br>
</h1>
An efficiently updatable neural network (NNUE, sometimes stylised as ƎUИИ) is a neural network-based evaluation function that runs efficiently on central processing units without a requirement for a graphics processing unit (GPU). Stockfish NNUE is a port of a shogi NNUE to Stockfish 10. NNUE was invented by Yu Nasu and introduced to computer shogi in 2018.

## Training Guide
### Generating Training Data
Use the "no-nnue.nnue-gen-sfen-from-original-eval" binary. The given example is generation in its simplest form. There are more commands. 
```
uci
setoption name Threads value x
setoption name Hash value y
setoption name SyzygyPath value path
isready
gensfen depth 12 loop 200000000
```
- `depth` is the searched depth per move, or how far the engine looks forward. This value is an integer.
- `loop` is the amount of positions generated. This value is also an integer.

Specify how many threads and how much memory you would like to use with the `x` and `y` values. The option SyzygyPath is not necessary, but if you would like to use it, you must first have Syzygy endgame tablebases on your computer.
This will save a file named "generated_kifu.bin" in the same folder as the binary. Once generation is done, move the file to a folder named "trainingdata" in the same directory as the binaries.
### Generating validation data
The process is the same as the generation of training data, except for the fact that you need to set loop to 1 million, because you don't need a lot of validation data. The depth should be the same as before or a little higher than the depth of the training data. 
```
uci
setoption name Threads value x
setoption name Hash value y
setoption name SyzygyPath value path
isready
gensfen depth 12 loop 1000000
```
Once generation is done, move the file to a folder named "validationdata" in the same directory as the binaries.
### Training a completely new network
Use the "halfkp_256x2-32-32.nnue-learn" binary. Create an empty folder named "evalsave" in the same directory as the binaries.
```
uci
setoption name SkipLoadingEval value true
setoption name Threads value x
isready
loop 100 batchsize 1000000 lambda 0.5 eta 01.0 newbob_decay 0.5 eval_save_interval 100000000 loss_output_interval 1000000 mirror_percentage 50 validation_set_file_name validationdata\generated_kifu.bin nn_batch_size 1000 eval_limit 3000
```
- `eta` is the learning rate.
- `lambda` is the amount of weight it puts to eval of learning data vs win/draw/loss results.

Nets get saved in the "evalsave" folder. Copy the net located in the "final" folder under the "evalsave" directory and move it into a new folder named "eval" under the directory with the binaries.
