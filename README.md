# About this repository

Code to generate results in "An algorithmic approach to reducing unexplained pain disparities in underserved populations". 

## Citation

If you use this code, please cite: 

Emma Pierson, David M. Cutler, Jure Leskovec, Sendhil Mullainathan, and Ziad Obermeyer. An algorithmic approach to reducing unexplained pain disparities in underserved populations. *Nature Medicine*, 2021.

If you use the OAI data, please cite: 

Michael C. Nevitt, David T. Felson, and Gayle Lester. The Osteoarthritis Initiative. 2006.

## Clone a branch

It is always best to clone via the command line instead of using VsCode built in 'tools' because Windows users should always clone using the `--config core.autocrlf=false` option, especially if working with someone elses code. The following command will clone the `joco-testing-2024.01.03` branch and save it to a `pd-joco-testing-2024.01.03` folder on your computer.

`git clone --branch joco-testing-2024.01.03 --config core.autocrlf=false https://github.com/kuhlaid/pain-disparities.git pd-joco-testing-2024.01.03`

## Regenerating results

1. **Setting up virtualenv**. Our code is run in a virtual environment using Python 3.5.2. You can set up the environment by using `virtualenv -p python3.5 YOUR_PATH_TO_VIRTUALENV`, activating the virtualenv via `source YOUR_PATH_TO_VIRTUALENV/bin/activate`, and then installing packages via `pip install -r requirements.txt`. Make sure the virtual environment is activated prior to running any of the steps below.  If you want to run `main_results_for_public_repo.ipynb` you will additionally need to run `python -m ipykernel install --user --name=knee` to install a kernel for the IPython notebook; make sure to use this kernel when running the notebook. 

Additionally, if you are going to run `image_processing.py`, our code makes use of the KneeLocalizer (repo)[https://github.com/MIPT-Oulu/KneeLocalizer] to crop knees in some of our initial experiments (not the final paper). For your convenience, we've provided a copy of this repo, since we made slight modifications to it to allow it to run on the OAI data. After setting up the virtualenv, please cd into the KneeLocalizer directory and run `python setup.py install`. Please let us know if you have any issues with this dependency, which is not essential to reproduce our analysis. 

2. **Data processing**. We provide the code needed to regenerate the processed data from raw OAI data (which can be downloaded at [https://nda.nih.gov/oai/](https://nda.nih.gov/oai/)). Data was processed on a computer with several terabytes of RAM and hundreds of cores. We do not know whether the OAI data format provided online will remain constant over time - eg, folder names may change - so please contact us if you have any questions. 

    - a. Process the original DICOM files into a pickle of numpy arrays. This can be done by running `python image_processing.py`. (We recommend running this in a screen session or similar because it takes a while). 
    - b. Write out the individual images as separate files because the original pickle is too large. This can be done by running 
        `python image_processing.py --normalization_method our_statistics --show_both_knees_in_each_image True --downsample_factor_on_reload None --write_out_image_data True --seed_to_further_shuffle_train_test_val_sets None --crop_to_just_the_knee False`. Again, we recommend running this in a screen session. Note this actually writes out four datasets, not three - train, val, test, and a blinded hold out set. As described in the paper, all exploratory analysis on the paper was performed using only the train, val, and test sets. However, for the final analysis, we retrained models on the train+test sets and evaluated on the blinded hold out set. The four datasets can be combined into three using the method `rename_blinded_test_set_files` in `constants_and_util.py`. 

3. **Set paths.** You will need to set paths suitable for your system in constants_and_util.py. Please see the "Please set these paths for your system" comment in `constants_and_util.py`, and the associated capitalized variables. 

4. **Training models.** Neural network experiments are performed using `python train_models.py EXPERIMENT_NAME`. (For valid experiment names, see the `train_one_model` method.) Running this script will train neural net models indefinitely (after a model is trained and saved, training for a new one begins) which is useful for ensembling models. Models in the paper were trained using four Nvidia XP GPUs. Specific experiments discussed in the results are: 

    - `train_best_model_continuous`: Trains models to predict pain using the best-performing config. 

    - `hold_out_one_imaging_site`: Trains the models using data from all but one imaging site to confirm results generalize across sites. 

    - `predict_klg`: Train the models to predict KLG rather than pain (using same config as in `train_best_model_continuous`) and show that our results are comparable to previous ones. 

    - `increase_diversity`: Assess the effect of altering the racial or socioeconomic diversity in the train dataset while keeping dataset size constant. 

5. **Analyzing models and generating figures for paper**. Once models have been run, figures and results in the paper can be reproduced by running `main_results_for_public_repo.ipynb`. Running this notebook takes a while (about a day) because of the number of bootstrap iterations, so we recommend running it in a screen session using, eg, `jupyter nbconvert --execute --ExecutePreprocessor.timeout=-1 --to notebook main_results_for_public_repo.ipynb`. A much faster approach is to run only the cells you need to reproduce the results of interest; alternately, you can reduce the number of bootstrap iterations. Note that running cells which call the class `non_image_data_processing.NonImageData` will require downloading the original non-image data from the OAI (but these files are much smaller and faster to process than the image files). 

## Files

**constants_and_util.py**: Constants and general utility methods. 

**non_image_data_processing.py**: Processes non-image data.

**image_processing.py**: Processes image data and combines with non-image data. 

**train_models.py**: Trains the neural network models used in analysis. 

**analysis.py**: Helper methods for analysis used in the paper. 

**main_results_for_public_repo.ipynb**: Generates the figures and numerical results in the paper using the methods in `analysis.py`. 

**requirements.txt**: Packages used in the virtualenv. 
