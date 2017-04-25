# pix2pix-tensorflow server

Host pix2pix-tensorflow models to be used with something like the [Image-to-Image Demo](https://affinelayer.com/pixsrv/).

This is a simple python server that serves models exported from `pix2pix.py --mode export`.  It can serve local models or use [Cloud ML](https://cloud.google.com/ml/) to run the model.

## Local

Using the [pix2pix-tensorflow Docker image](https://hub.docker.com/r/affinelayer/pix2pix-tensorflow/):

```sh
# export a model to upload
python ../tools/dockrun.py python export-example-model.py --output_dir models/example
# process an image with the model using local tensorflow
python ../tools/dockrun.py python process-local.py \
    --model_dir models/example \
    --input_file static/facades-input.png \
    --output_file output.png
# run local server
python ../tools/dockrun.py python --port 8000 serve.py --local_models_dir models
# test the local server
python process-remote.py \
    --input_file static/facades-input.png \
    --url http://localhost:8000/example \
    --output_file output.png
```

If you open [http://localhost:8000/](http://localhost:8000/) in a browser, you should see an interactive demo, though this expects the server to be hosting the exported models available here:

- [edges2shoes](https://mega.nz/#!HtYwAZTY!5tBLYt_6HFj9u2Kxgp4-I36O4EV9r3bDP44ztX3qesI)
- [edges2handbags](https://mega.nz/#!Clg3EaLA!YW2jfRHvwpJn5Elww_wM-f3eRzKiGHLw-F4A3eQCceI)
- [facades](https://mega.nz/#!f1ZjmZoa!mCSxFRxt1WLBpNFsv5raoroEigxomDVpdi40aOG1KMc)

Extract those to the models directory and restart the server to have it host the models.

## Cloud ML

For this you'll want to generate a service account JSON file from https://console.cloud.google.com/iam-admin/serviceaccounts/project (select "Furnish a new private key").  If you are already logged in with the gcloud SDK, the script will auto-detect credentials from that if you leave off the `--credentials` option.

```sh
# upload model to google cloud ml
python ../tools/dockrun.py python upload-model.py \
    --bucket your-models-bucket-name-here \
    --model_name example \
    --model_dir models/example \
    --credentials service-account.json
# process an image with the model using google cloud ml
python ../tools/dockrun.py python process-cloud.py \
    --model example \
    --input_file static/facades-input.png \
    --output_file output.png \
    --credentials service-account.json
```

## Google Cloud Platform

Assuming you have gcloud and docker setup:

```sh
export GOOGLE_PROJECT=<project name>
export GOOGLE_CREDENTIALS="$(cat <path to service-account.json>)"
# build image
# make sure models are in a directory called "models" in the current directory
sudo docker build --rm --tag us.gcr.io/$GOOGLE_PROJECT/pix2pix-server .
# test image locally
sudo docker run --publish 8080:8080 --rm --name server us.gcr.io/$GOOGLE_PROJECT/pix2pix-server python -u serve.py \
    --port 8080 \
    --local_models_dir models
python process-remote.py \
    --input_file static/facades-input.png \
    --url http://localhost:8080/example \
    --output_file output.png
# publish image to private google container repository
gcloud docker -- push us.gcr.io/$GOOGLE_PROJECT/pix2pix-server
# setup server
# need to change the launch arguments for google_compute_instance.pix2pix-singleton
# to use local models instead of cloud models if desired
python ../tools/dockrun.py terraform plan -var "GOOGLE_PROJECT=$GOOGLE_PROJECT" -target google_compute_instance.pix2pix-singleton
python ../tools/dockrun.py terraform apply -var "GOOGLE_PROJECT=$GOOGLE_PROJECT" -target google_compute_instance.pix2pix-singleton
```