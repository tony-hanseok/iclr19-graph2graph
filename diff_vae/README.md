# Variational Junction Tree Encoder-Decoder

The folder contains the training and testing scripts for variational junction tree encoder-decoder. The trained model is saved in `models/` for all datasets. Sample translations of the test set are provided in `results/` folder. 

Please make sure that this repo can be found by the system by running
```
export PYTHONPATH=$path_to_repo/iclr19-graph2graph
```

## Preprocessing
For fast training, we need to first preprocess the training data. For instance, you can preprocess the training set of logp04 task using:
```
python ../scripts/preprocess.py --train ../data/logp04/train_pairs.txt --ncpu 8
mkdir processed
mv tensor* processed
```
This will put all binary files into `processed/` folder. To preprocess all the datasets, run
```
bash preprocess.sh
```
This will create four folders in `../data/*/processed` for four tasks respectively.

## Vocabulary Extraction (Optional)
The fragment vocabulary files have already been provided in `data/` for all datasets. If you are working on your own dataset, you will need to extract the corresponding fragment vocabulary. Supposed your molecule file is `mols.txt`, which has a SMILES string in each line, then you can run the following script for vocabulary extraction:
```
python ../fast_jtnn/mol_tree.py < mols.txt
```

## Training
Suppose the preprocessed data of logp04 is saved in `processed/` folder. You can train our model on the logp04 task by
```
mkdir -p newmodels/logp04
python vae_train.py --train processed/ --vocab ../data/logp04/vocab.txt --save_dir newmodels/logp04 \
--hidden_size 330 --rand_size 8 --epoch 12 --anneal_rate 0.8 | tee newmodels/logp04/LOG
```
Specifically, `--hidden_size 330` sets the hidden state dimension to be 300 and `--rand_size 8` sets the latent code dimension to be 8. `--epoch 12 --anneal_rate 0.8` means the model will be trained for 12 epochs, with learning rate annealing rate 0.8. The model checkpoints are saved in `newmodels/logp04`.

The hyperparameters of our experiments in the paper are provided in `train.sh`. Our models with the best validation performance are provided in `models/` folder (e.g., `models/logp04/model.iter-5` for the logp04 model). After you have preprocessed all four datasets, you can run `bash train.sh` to train our model on all datasets.

## Validation

## Testing
For example, you can test the above model on the logp04 task by running
```
python decode.py --test ../data/logp04/test.txt --vocab ../data/logp04/vocab.txt --model models/logp04/model.iter-5 | python ../scripts/logp_score > results.logp04
python logp_analyze.py --delta 0.4 < results.logp04
```
You can test our models on all four tasks by running
```
bash decode_and_eval.sh
```
or equivalently `bash eval.sh`. This should print the model performance as follows:
```
```