# Baselines recap for the probablistic particles project


## Argoverse 
All code can be found on [this gitlab repo](https://gitlab.nautilus.optiputer.net/Sophiaphia/argoversemfp). It also contains all the processed argoverse data (in total ~2.3G), designed to be a self-contained repo for servers to pull and run training. 

If you have Nautilus & kubernetics set up, you can run a training by: 

`kubectl create -f argo-mfp-6-quant.yaml`

with the yaml file in this directory. `quant` meand quantil loss, and `nll` means nll loss. The only difference is which `multiple_futures_prediction/train` stripts it calls from the repo. 

### Changes made from the original mfp code

I kept most of the original code as reference; anything with `_argoverse` at the end is added. 

**data pre-processing** `dataset_argoverse.ipynb` contains the code to process argoverse data into a `.mat` file for the dataloader. The repo already contains these processed mat files, so no need to run. In case you want to try it out, beware that it can get quite memory-intense.

**dataloader**  `multiple_futures_prediction/dataset_argoverse.py` There are multiple pre-processing functions here. `getHistory` performs no normalization, `getHistoryTranslate` shifts everything to the coordinate system of the reference vehicle (`refVehId`), and `getHistoryRotate` performs translation, and then rotates the trajectories around 0. Same naming convention applyes to `getFuture`. These functions are called in `__getitem__`. 

Another caveate here is there are some hard-coded placeholders. The neighbours dictionary is hard-coded (only looks at AV and Agent as of now), neighbours_mask is zeros, and context is none. Each batch only contains 2 trajectories (2 vehicels from the same scene.) I'm not sure which piece is detereorating performance by how much.

**metrics** `multiple_futures_prediction/my_utils.py` The quantile losses (`quantile_loss` and `quantile_loss_multimodes`) at the top of the file. Without `_eval` it returns the average over timesteps, with `_eval` it returns the sum and the count. `multimodes` simply takes the min accross all Gaussians, which is not great. I will implement proper scoring for GMM soon.

**eval** `multiple_futures_prediction/eval_argoverse.py` You can run this class by running `python -m multiple_futures_prediction.cmd.eval_argoverse_cmd --config multiple_futures_prediction/configs/mfp1_argoverse.gin` where you replace the config file with the correct gin config that you used to train. Here there is only the 1 mode and 5 mode difference. Eval calls `eval_all`, which prints out all the metrics (ADE, FDE, NLL, MIS).

You can also do eval interactively + make visualizations in `vis.ipynb`. I've included a bunch of train models in this repo's folder `checkpoints`. 

**training** Made loss customizable via gin config.


### Might be helpful

#### Kubernetics job debug commands 

```
kubectl create -f mfp-job.yaml 
kubectl describe jobs/mfp-job

kubectl describe pods mfp-job-spmwp

kubectl logs mfp-job-lzr2h  -c init-clone-repo
kubectl logs --tail=20 mfp-job-xtxsv

kubectl cp <file-spec-src> <file-spec-dest>
kubectl delete jobs/mfp-job
```
