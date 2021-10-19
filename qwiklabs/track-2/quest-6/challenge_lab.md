<h1 align="center">Task 1: Start a JupyterLab Notebook instance</h1>

<h3>Using the cloud console</h3>

<h3>Create a bucket</h3>

Naviagtion menu-> Cloud Storage-> Browser-> Create bucket-> Set the name of your bucket as your Project ID-> Click Create

Navigation Menu->AI Platform->Notebooks->New Instance

Select TensorFlow Enterprise 2.3 Without GPUs

Click Create

Once it get loaded, click Open Juputerlab.

<h1 align="center">Task 2: Download the Challenge Notebook
</h1>

Select Terminal

<h3>Clone the repo</h3>
git clone https://github.com/GoogleCloudPlatform/training-data-analyst

* Go to the enclosing folder: training-data-analyst/quests/dei
* Open the notebook file what-if-tool-challenge.ipynb.
* Download and import the dataset hmda_2017_ny_all-records_labels.

Run the first eight cells to get and test the data.

<h1 align="center">Task 3: Build and train your models</h1>

Run the ninth cell to import the required libraries

Add this code to train your first model
```
model = Sequential()
model.add(layers.Dense(50, input_shape=(input_size,), activation='relu'))
model.add(layers.Dense(10, activation='relu'))
model.add(layers.Dense(1, activation='sigmoid'))
model.compile(loss='mean_squared_error', optimizer='adam', metrics=['accuracy'])
model.fit(train_data, train_labels, epochs=10, batch_size=2048, validation_split=0.1)
```

For your second model,

```
limited_model = Sequential()
limited_model.add(layers.Dense(50, input_shape=(input_size,), activation='relu'))
limited_model.add(layers.Dense(10, activation='relu'))
limited_model.add(layers.Dense(1, activation='sigmoid'))
limited_model.compile(loss='mean_squared_error', optimizer='adam', metrics=['accuracy'])
limited_model.fit(limited_train_data, limited_train_labels, epochs=10, batch_size=2048, validation_split=0.1)
```

<h1 align="center">Task 4: Deploy the models to AI Platform</h1>

Replace #TODO with your Project ID.

Remeber the region, this will be used in the next steps.

Run the three cells.

<h2>Create your first AI Platform model: saved_complete_model</h2>

Navigation Menu->AI Platform->Models

Select region same as in the previous step where you have deployed.

Click Create model
* Name: complete_model
* Region: same as in the previous step.

Click Create.

After it gets created. Go to model.

Click Create version

* Model Name = complete_model
* Version Name = v1
* Python version = 3.7
* Framework = TensorFlow
* Framework version = 2.3.1
* ML Runtime version = 2.3

<h2>Create your second AI Platform model: saved_limited_model</h2>

Repeat the above steps.

* Model Name = limited_model
* Version Name = v1
* Python version = 3.7
* Framework = TensorFlow
* Framework version = 2.3.1
* ML Runtime version = 2.3