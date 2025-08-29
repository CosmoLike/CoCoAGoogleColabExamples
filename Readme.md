# Google Colab Examples (CoCoA)

This repository contains examples that show how to run CoCoA, the [Cobaya-Cosmolike Joint Architecture](https://github.com/CosmoLike/cocoa), within Google Colab.

## :interrobang: FAQ: How can users run Cocoa on Google Colab? <a name="overview_google_colab"></a>

[Google Colab](https://colab.research.google.com/) provides a convenient platform for users to run MCMCs, likelihood minimizations, and profiles, as long as Machine-Learning Emulators are used to compute the data vectors. In the repository [CoCoAGoogleColabExamples](https://github.com/CosmoLike/CoCoAGoogleColabExamples), we provide a few examples along with explanatory notes. 

Installing Cocoa requires time and also strains our limited Git-LFS quota, which is especially relevant given that **the entire local drive is wiped when a Colab notebook is disconnected**. To prevent this problem, we provide instructions on how to save and load Cocoa immediately after the initial installation. 

There are a few differences users should be aware of when running Cocoa on Google Colab.

  - Running Collab Notebook for the first time

    - **Cell :one:**: Connect the notebook to your Google Drive account (will be important later)

          from google.colab import drive
          drive.mount('/content/drive')

    - **Cell 2️⃣**: Install Miniforge (Similar to our documentation in section [FAQ: How can users install Conda?](#overview_miniforge))

          %%bash
          export CONDA_DIR="/content/conda"
          mkdir "${CONDA_DIR:?}"
          curl -L -O "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh"
          /bin/bash Miniforge3-$(uname)-$(uname -m).sh -f -b -p "${CONDA_DIR:?}"
          /bin/bash
          source $CONDA_DIR/etc/profile.d/conda.sh \
                && conda config --set auto_update_conda false \
                && conda config --set show_channel_urls true \
                && conda config --set auto_activate_base false \
                && conda config --prepend channels conda-forge \
                && conda config --add allowlist_channels conda-forge \
                && conda config --set channel_priority strict \
                && conda init bash
          source ~/.bashrc

    - **Cell 3️⃣**: Install Conda cocoa env (similar to our documentation in section [Installation of core packages](#required_packages_conda))

          %%bash
          source "/content/conda/etc/profile.d/conda.sh"
          conda activate lockenv
          conda-lock install -n cocoa cocoapy310-linux.yml
          conda activate cocoa 
          ln -s "${CONDA_PREFIX}"/bin/x86_64-conda_cos6-linux-gnu-gcc "${CONDA_PREFIX}"/bin/gcc
          ln -s "${CONDA_PREFIX}"/bin/x86_64-conda_cos6-linux-gnu-g++ "${CONDA_PREFIX}"/bin/g++
          ln -s "${CONDA_PREFIX}"/bin/x86_64-conda_cos6-linux-gnu-gfortran "${CONDA_PREFIX}"/bin/gfortran
          ln -s "${CONDA_PREFIX}"/bin/x86_64-conda-linux-gnu-gcc-ar "${CONDA_PREFIX}"/bin/gcc-ar
          ln -s "${CONDA_PREFIX}"/bin/x86_64-conda-linux-gnu-gcc-ranlib "${CONDA_PREFIX}"/bin/gcc-ranlib
          git-lfs install

    - **Cell 4️⃣**: Clone CoCoA (similar to our documentation in section [Installation and Compilation of external modules](#cobaya_base_code))

          %%bash
          source "/content/conda/etc/profile.d/conda.sh"
          conda activate cocoa                                  
          git clone https://github.com/CosmoLike/cocoa.git --branch v4.0 cocoa # users can adjust this line

    - **Cell 5️⃣**: run `setup_cocoa.sh`

           %%bash
           source "/content/conda/etc/profile.d/conda.sh" 
           conda activate cocoa
           cd ./cocoa/Cocoa/
           source setup_cocoa.sh

    - **Cell 6️⃣**: run `compile_cocoa.sh`

          %%bash
          source "/content/conda/etc/profile.d/conda.sh"
          conda activate cocoa
          cd ./cocoa/Cocoa/
          source compile_cocoa.sh

    - **Cell 7️⃣**: Save CoCoA on Drive (does not work with local runtime)

          %%bash
          DEST="/content/drive/MyDrive/ColabBackups"
          ARCHIVE="$DEST/colab_basic_cocoa.tar.gz"
          if [[ -f "$ARCHIVE" ]]; then
            echo "Backup already exists: $ARCHIVE — skipping."
            exit 0
          fi
          mkdir -p "$DEST"
          tar -czf "$DEST/colab_basic_cocoa.tar.gz" \
            --exclude='/content/drive' \
            --exclude='**/__pycache__' \
            --exclude='**/.ipynb_checkpoints' \
            /content
          echo "Created: $ARCHIVE"

  - Running Collab Notebook with CoCoA pre-compiled, loaded from Drive (does not work with local runtime)

    - **Cell :one:**: Connect the notebook to your Google Drive account

          from google.colab import drive
          drive.mount('/content/drive')

    - **Cell 2️⃣**: Load CoCoA from Drive

          %%bash
          DEST="/content/drive/MyDrive/ColabBackups"
          ARCHIVE="$DEST/colab_basic_cocoa.tar.gz"
          SENTINEL="/content/conda/etc/profile.d/conda.sh"  # exists when your env is restored
          if [[ -e "$SENTINEL" ]]; then
            echo "Found $SENTINEL — environment already restored. Skipping untar."
            exit 0
          fi
          test -f "$ARCHIVE"
          ARCHIVE="/content/drive/MyDrive/ColabBackups/colab_basic_cocoa.tar.gz"
          tar -xzf "$ARCHIVE" -C /

> [!Note]
> From now on, users must start every subsequent shell with 
>
>        %%bash
>        source "/content/conda/etc/profile.d/conda.sh"
>        conda activate cocoa`
>        cd ./cocoa/Cocoa/
>        source start_cocoa.sh

## :interrobang: FAQ: How can users save checkpoints on Google Colab? <a name="overview_google_colab2"></a>

  One of the biggest challenges in working with Google Colab is the fact that the entire local drive is erased when a notebook is disconnected. Not reserving time to copy the `/content` folder to the user's Google Drive, an expensive operation, can result in 24 hours of lost computation. To prevent such a catastrophe, our examples contain several blocks that create **checkpoints** after computationally intensive cells. 

**This solution is not valid when running Colab with local runtime** (see [Google documentation](https://research.google.com/colaboratory/local-runtimes.html) for additional information on how to link notebooks to local resources). The good news here is that local storage is persistent, so there is no need to create backups on Google Drive.
  
  - How to create a checkpoint? 

      - **Step :one:**: Connect the notebook to your Google Drive account
      
            from google.colab import drive
            drive.mount('/content/drive')
    
      - **Step 2️⃣**: Compress and copy the `/content` folder from the local disk to the user's Drive
    
            %%bash
            ROOT="colab_name_notebook"
            DEST="/content/drive/MyDrive/ColabBackups"
            mkdir -p "$DEST"
            ARCHIVE="$DEST/$ROOT_$(date +%F_%H-%M).tar.gz"
            tar -czf "$ARCHIVE" \
                --exclude='/content/drive' \
                --exclude='**/__pycache__' \
                --exclude='**/.ipynb_checkpoints' \
                /content
            echo "Created: $ARCHIVE"

  - How to load a checkpoint?

      - **Step :one:**: Connect the notebook to your Google Drive account
      
            from google.colab import drive
            drive.mount('/content/drive')
    
      - **Step 2️⃣**: Decompress and copy the `/content` folder from the user's Drive to the local disk (set `ARCHIVE` to the appropriate checkpoint file)
    
            %%bash
            SENTINEL="/content/conda/etc/profile.d/conda.sh"  # exists when your env is restored
            if [[ -e "$SENTINEL" ]]; then
              echo "Found $SENTINEL — environment already restored. Skipping untar."
              exit 0
            fi
            ARCHIVE="CHECKPOINT_FILE"
            test -f "$ARCHIVE"
            tar -xzf "$ARCHIVE" -C /
