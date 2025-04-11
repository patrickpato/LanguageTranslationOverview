### LANGUAGE TRANSLATION WITH OPENNMT-PY
* This repository contains the data preparation and best-model configs for language translation between English and 4 Bantu languages, Swahili, Luganda, Shona and Tsonga.
* The objective of this work is to investigate the performance of Neural Machine Translation in low-resources language translation tasks.
* The codes were all executed in Google Colab environments.
* Details on the environment's resources will be provided in the end of this README document.

  ## Data Preparation
   # Pre-requisites
  1. Ensure you have parralel corpuses of data in txt format e.g Shona corpus and corresponding English corpus.
  2. The txts should be aligned, that is line 1 of Shona corpus should correspond to line 1 of English corpus.
  3. Both file should have the same number of lines.
  4. Ensure that you have connected your Google drive to the colab environment as the tasks are quite disk-space intensive. 
 
  # Preparation Steps:
  1. Create a NeuralMT directory in your Colab environment, navigate into the directory and clone the repository below to your working environment in Colab:
     ```bash
      !mkdir -p NeuralMT
      %cd NeuralMT
      !git clone https://github.com/ymoslem/MT-Preparation.git
      ```
  2. Install the required packages from the repository above:
    ```bash
     !pip3 install -r MT-Preparation/requirements.txt
    ```
  4. Implement subwording using Byte-Pair-Encoding as implemented in the repository above:
     ```bash
     !python3 MT-Preparation/subwording/1-train_bpe.py /content/drive/MyDrive/LanguageTranslationData_New/BackTranslation/LugEng/English_Luganda_Lug.txt  /content/drive/MyDrive/LanguageTranslationData_New/BackTranslation/LugEng/English_Luganda_Eng.txt
     ```
     ** This will create four files, source.model, target.model, target.model and source.vocab in the /content/NeuralMT directory that we are working from.
     ** source.model and source.vocab are for the source data source, which in this case is English while target.model and target.vocab are for the target language, i.e Bantu language.

  5. Copy the .model and .vocab files to your root directory containing the language pair to be translated. This step will ensure that the file is present in your Google drive even after refreshing the Colab environment:
  6.   ```bash
       !mv /content/NeuralMT/source.model /content/drive/MyDrive/LanguageTranslationData_New/BackTranslation/LugEng/
       !mv /content/NeuralMT/source.vocab /content/drive/MyDrive/LanguageTranslationData_New/BackTranslation/LugEng/
       ```
     
  7. Implement subwording using the source model and target model obtained from above. The source model should point to the source corpus and the target model should point to the target corpus.
  8. ```bash 
     !python3 MT-Preparation/subwording/2-subword.py /content/drive/MyDrive/LanguageTranslationData_New/BackTranslation/LugEng/source.model /content/drive/MyDrive/LanguageTranslationData_New/BackTranslation/LugEng/target.model /content/drive/MyDrive/LanguageTranslationData_New/BackTranslation/LugEng/English_Luganda_Lug.txt /content/drive/MyDrive/LanguageTranslationData_New/BackTranslation/LugEng/English_Luganda_Eng.txt
     ```
    
  9. Finally, split the subworded data into train, test and dev sets using the command below (pass the number of lines to be used for testing and validation and the script will automatically compute the number of training lines based on the size of the data):
   ```bash
     !python3 MT-Preparation/train_dev_split/train_dev_test_split.py 12000 2400 /content/drive/MyDrive/LanguageTranslationData_New/BackTranslation/LugEng/English_Luganda_Lug.txt.subword /content/drive/MyDrive/LanguageTranslationData_New/BackTranslation/LugEng/English_Luganda_Eng.txt.subword
     *
    ```

  ## Language Translation Implementation in OpenNMT-PY
  * This part should be performed in a new colab environment altogether.
  * Ensure you load your Google drive to the Colab environment to load the data from the data preparation step above:
    ```bash
    from google.colab import drive
    drive.mount('/content/drive')
    ```
  * Install required libraries as below:
    ```bash
    !apt-get -y install protobuf-compiler python-pil python-lxml
    !pip install openNMT-py
    ```
  * Set up the OpenNMT-PY Model configuration in a conf.yaml file. Among others, this file contains (emphasis on the model configs has been outlined in the paper):
    
      1. The path to the train and test data sets from the previous data preparation step. These are contained in the path_src and path_tgt parameters.
      2. The paths to the vocab files generated from OpenNMT build vocab task that is executed prior to model training.
      3. The tokenization models generated from the BPE task implemented in the Data Preparation process.
      4. Model configs that have been optimized.
    
  * Build the source and target vocabulary based on the configuration.yaml generated above:
    ```bash
    !onmt_build_vocab -config /content/config.yaml -n_sample -1 -num_threads 2
    ```
  * Train the model using the command:
    ```bash
    !onmt_train -config /content/config.yaml
    ```
  * Run the translation on the validation dataset as below:
    ```bash
    !onmt_translate -model /content/models/model.engluganda_step_6000.pt -src /content/drive/MyDrive/LanguageTranslationData_New/EngLuganda_Kimera_Benchmark/English_Luganda_Eng.txt.subword.dev -output /content/drive/MyDrive/LanguageTranslationData_New/EngLuganda_Kimera_Benchmark/Luganda_TranslatedV11.translated

    ```

  ## Model Evaluation
  * In your Colab environment, navigate back to the root environment and run the commands below to de-subword the translated file from the step above and the validation subword file created in the data preparation step by running the commands:
    ```bash
    %cd  ../
    !ls
    !python3 nmt/MT-Preparation/subwording/3-desubword.py /content/drive/MyDrive/LanguageTranslationData_New/EngLuganda_Kimera_Benchmark/target.model  /content/drive/MyDrive/LanguageTranslationData_New/EngLuganda_Kimera_Benchmark/Luganda_TranslatedV11.translated
    !python3 nmt/MT-Preparation/subwording/3-desubword.py /content/drive/MyDrive/LanguageTranslationData_New/EngLuganda_Kimera_Benchmark/target.model /content/drive/MyDrive/LanguageTranslationData_New/EngLuganda_Kimera_Benchmark/English_Luganda_Lug.txt.subword.dev
     ```
   * Install the sacrebleu library to be used in the BLEU score computation and obtain the BLEU score from the de-subworded files using the commands:
     ```bash
      !wget https://raw.githubusercontent.com/ymoslem/MT-Evaluation/main/BLEU/compute-bleu.py
      !pip3 install sacrebleu
      !python3 compute-bleu.py /content/drive/MyDrive/LanguageTranslationData_New/EngLuganda_Kimera_Benchmark/English_Luganda_Lug.txt.subword.dev.desubword /content/drive/MyDrive/LanguageTranslationData_New/EngLuganda_Kimera_Benchmark/Luganda_TranslatedV11.translated.desubword
     ```
   * Compute the METEOR score using the following Python code:
     ```python
     from nltk.translate.meteor_score import meteor_score, single_meteor_score
     import nltk
     # Download necessary NLTK data
     nltk.download('wordnet')
     nltk.download('punkt')
      
     # Function to read the content of a text file
     def read_file(file_path):
       with open(file_path, 'r', encoding='utf-8') as file:
         return file.read()
      
      # Tokenize the text
      def tokenize(text):
          return nltk.word_tokenize(text)
     
      # Function to read the content of a text file
      def read_file(file_path):
      with open(file_path, 'r', encoding='utf-8') as file:
        return file.read()

      # Paths to your text files
      file1_path = '/content/drive/MyDrive/LanguageTranslationData_New/EngLuganda_Kimera_Benchmark/English_Luganda_Lug.txt.subword.dev.desubword'
      file2_path = '/content/drive/MyDrive/LanguageTranslationData_New/EngLuganda_Kimera_Benchmark/Luganda_TranslatedV11.translated.desubword'

      # Read the contents of the files
      reference = read_file(file1_path)
      hypothesis = read_file(file2_path)
      
      # Tokenize the texts
      reference_tokens = tokenize(reference)
      hypothesis_tokens = tokenize(hypothesis)
      
      # Calculate the METEOR score
      score = meteor_score([reference_tokens], hypothesis_tokens)
      
      print(f'The METEOR score between the two files is: {score}')
     ```

   ## Environment Required
  * OpenNMT-Py for this task was executed using Colab Pro environment
  * Hardware accelerator used was A100 GPU. This is based on the volume of the data being translated. 
    
  
