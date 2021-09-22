# tensorflow-mnist-model

The `preprocessing.py` and `train.py` processes and trains a mnist model using the quay.io/ibmtechgarage/tensorflow-model-train image. 

### preprocessing.py

The preprocessing script takes in a single argument, `data_dir`. It downloads and preprocesses the data and saves the `pickled` version in `data_dir`. In production code, this would probably be the directory where TFRecords are stored, for example.


## Testing

### locally with docker
```
docker run --name tensorflow-mnist -v containers:/var/lib/containers --rm -it quay.io/ibmtechgarage/tensorflow-model-train
```
Copy Files into container
```
docker cp environment.yml tensorflow-mnist:/tmp/ 
docker cp preprocessing.py tensorflow-mnist:/tmp/ 
docker cp train.py tensorflow-mnist:/tmp/ 
docker cp constants.py tensorflow-mnist:/tmp/ 
docker cp build-script.sh tensorflow-mnist:/tmp/
```
Exec build-script
```
docker exec tensorflow-mnist sh -c ./build-script.sh
```