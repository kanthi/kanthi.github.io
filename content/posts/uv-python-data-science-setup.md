+++
title = "UV + Python: The Ultimate Data Science and AI Development Environment Setup"
date = "2025-07-28T14:00:00+05:30"
author = ""
draft = true
authorTwitter = "" #do not include @
cover = ""
tags = ["uv", "python", "data-science", "ai", "machine-learning", "jupyter", "pandas", "pytorch", "tensorflow", "development"]
keywords = ["uv python", "data science setup", "ai development", "machine learning environment", "jupyter lab", "python data science", "pytorch setup", "tensorflow setup"]
description = "Complete guide to setting up a blazing-fast Python data science and AI development environment using UV, with Jupyter, PyTorch, TensorFlow, and all essential data science tools."
showFullContent = false
readingTime = false
hideComments = false
+++

Data science and AI development require robust, reproducible environments with complex dependencies that can be challenging to manage. Enter UV - the lightning-fast Python package manager that's revolutionizing how data scientists and AI engineers set up and maintain their development environments. This comprehensive guide will show you how to create a professional-grade data science setup using UV.

## Why UV is Perfect for Data Science and AI

### 1. **Lightning-Fast Package Installation**
Data science packages like NumPy, Pandas, PyTorch, and TensorFlow have complex dependencies. UV's 10-100x speed improvement over pip means you spend less time waiting and more time analyzing data.

### 2. **Superior Dependency Resolution**
AI/ML packages often have conflicting dependencies. UV's advanced resolver handles these conflicts better than traditional tools, reducing environment breakage.

### 3. **Reproducible Environments**
UV's lockfiles ensure your data science environment is exactly reproducible across different machines and team members.

### 4. **Multiple Python Versions**
Different AI frameworks may require different Python versions. UV makes managing multiple Python installations seamless.

### 5. **Project Isolation**
Each data science project can have its own isolated environment with specific package versions.

## System Prerequisites

### Base System Setup

```bash
# Ubuntu/Debian
sudo apt update && sudo apt upgrade -y
sudo apt install -y build-essential curl git wget software-properties-common

# Install system-level dependencies for data science
sudo apt install -y \
    python3-dev \
    python3-pip \
    libhdf5-dev \
    libnetcdf-dev \
    libopenblas-dev \
    liblapack-dev \
    gfortran \
    libjpeg-dev \
    libpng-dev \
    libfreetype6-dev \
    pkg-config \
    libffi-dev \
    libssl-dev

# For GPU support (NVIDIA)
# Add NVIDIA package repositories
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-keyring_1.0-1_all.deb
sudo dpkg -i cuda-keyring_1.0-1_all.deb
sudo apt update
sudo apt install -y cuda-toolkit-12-3

# macOS
brew install python@3.11 hdf5 netcdf openblas lapack jpeg libpng freetype pkg-config
```

### Install UV

```bash
# Install UV
curl -LsSf https://astral.sh/uv/install.sh | sh

# Add to shell profile
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

# Verify installation
uv --version
```

## Setting Up Python Versions for Data Science

### Install Multiple Python Versions

```bash
# Install Python versions commonly used in data science
uv python install 3.9    # Stable, widely supported
uv python install 3.10   # Good balance of features and compatibility
uv python install 3.11   # Latest stable with performance improvements
uv python install 3.12   # Cutting edge

# List installed versions
uv python list

# Set default Python version
uv python pin 3.11
```

### Create Data Science Project Structure

```bash
# Create main data science workspace
mkdir ~/datascience-workspace
cd ~/datascience-workspace

# Initialize UV project
uv init data-science-env --python 3.11
cd data-science-env

# Create project structure
mkdir -p {data/{raw,processed,external},notebooks,src,models,reports,config,tests}

# Create directory structure
tree
# data-science-env/
# ├── data/
# │   ├── raw/          # Original, immutable data
# │   ├── processed/    # Cleaned, transformed data
# │   └── external/     # External datasets
# ├── notebooks/        # Jupyter notebooks
# ├── src/             # Source code modules
# ├── models/          # Trained models
# ├── reports/         # Analysis reports
# ├── config/          # Configuration files
# └── tests/           # Unit tests
```

## Core Data Science Package Installation

### Essential Data Science Stack

```bash
# Core data manipulation and analysis
uv add pandas numpy scipy matplotlib seaborn plotly

# Scientific computing
uv add scikit-learn statsmodels

# Jupyter ecosystem
uv add jupyter jupyterlab ipywidgets nbconvert

# Data visualization
uv add bokeh altair plotly-dash streamlit

# Development tools
uv add --dev pytest black flake8 mypy pre-commit

# Install all dependencies
uv sync
```

### Advanced Analytics Packages

```bash
# Time series analysis
uv add prophet fbprophet statsforecast

# Natural Language Processing
uv add nltk spacy transformers datasets tokenizers

# Computer Vision
uv add opencv-python pillow scikit-image

# Big Data tools
uv add dask polars pyarrow

# Database connectivity
uv add sqlalchemy psycopg2-binary pymongo redis

# API and web scraping
uv add requests beautifulsoup4 scrapy fastapi

# Geospatial analysis
uv add geopandas folium

# Financial analysis
uv add yfinance quantlib-python
```

## Machine Learning Framework Setup

### PyTorch Installation

```bash
# CPU-only PyTorch (for development/testing)
uv add torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cpu

# GPU-enabled PyTorch (CUDA 12.1)
uv add torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121

# PyTorch ecosystem
uv add torchtext torchmetrics pytorch-lightning

# Verify PyTorch installation
uv run python -c "import torch; print(f'PyTorch version: {torch.__version__}'); print(f'CUDA available: {torch.cuda.is_available()}')"
```

### TensorFlow Installation

```bash
# TensorFlow (automatically detects GPU)
uv add tensorflow tensorflow-datasets tensorflow-probability

# TensorFlow ecosystem
uv add keras tensorboard

# Verify TensorFlow installation
uv run python -c "import tensorflow as tf; print(f'TensorFlow version: {tf.__version__}'); print(f'GPU devices: {tf.config.list_physical_devices(\"GPU\")}')"
```

### Hugging Face Ecosystem

```bash
# Transformers and datasets
uv add transformers datasets accelerate

# Additional Hugging Face tools
uv add tokenizers evaluate diffusers

# Audio processing
uv add librosa soundfile

# Test Hugging Face setup
uv run python -c "from transformers import pipeline; print('Hugging Face Transformers ready!')"
```

## Jupyter Lab Configuration

### Enhanced Jupyter Setup

```bash
# Install Jupyter extensions
uv add jupyterlab-git jupyterlab-lsp python-lsp-server

# Install additional kernels for different environments
uv add ipykernel

# Register kernel
uv run python -m ipykernel install --user --name=data-science-env --display-name="Data Science (UV)"

# Configure Jupyter Lab
mkdir -p ~/.jupyter

# Create Jupyter config
cat > ~/.jupyter/jupyter_lab_config.py << 'EOF'
c.ServerApp.ip = '0.0.0.0'
c.ServerApp.port = 8888
c.ServerApp.open_browser = False
c.ServerApp.token = ''
c.ServerApp.password = ''
c.ServerApp.allow_root = True
c.ServerApp.notebook_dir = '/home/user/datascience-workspace'

# Enable extensions
c.LabApp.extensions_in_dev_mode = True
EOF
```

### Start Jupyter Lab

```bash
# Start Jupyter Lab
uv run jupyter lab

# Or with specific configuration
uv run jupyter lab --ip=0.0.0.0 --port=8888 --no-browser --allow-root
```

## Project Configuration Files

### pyproject.toml Configuration

```toml
# pyproject.toml
[project]
name = "data-science-env"
version = "0.1.0"
description = "Data Science and AI Development Environment"
authors = [
    {name = "Your Name", email = "your.email@example.com"}
]
dependencies = [
    "pandas>=2.0.0",
    "numpy>=1.24.0",
    "scipy>=1.10.0",
    "matplotlib>=3.7.0",
    "seaborn>=0.12.0",
    "scikit-learn>=1.3.0",
    "jupyter>=1.0.0",
    "jupyterlab>=4.0.0",
    "plotly>=5.15.0",
    "torch>=2.0.0",
    "tensorflow>=2.13.0",
    "transformers>=4.30.0",
    "datasets>=2.14.0",
]
requires-python = ">=3.9"

[project.optional-dependencies]
dev = [
    "pytest>=7.0.0",
    "black>=23.0.0",
    "flake8>=6.0.0",
    "mypy>=1.0.0",
    "pre-commit>=3.0.0",
    "ipykernel>=6.0.0",
]
nlp = [
    "nltk>=3.8.0",
    "spacy>=3.6.0",
    "transformers>=4.30.0",
    "datasets>=2.14.0",
]
cv = [
    "opencv-python>=4.8.0",
    "pillow>=10.0.0",
    "scikit-image>=0.21.0",
]
bigdata = [
    "dask>=2023.7.0",
    "polars>=0.18.0",
    "pyarrow>=12.0.0",
]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.uv]
dev-dependencies = [
    "jupyter-lsp>=2.0.0",
    "python-lsp-server>=1.7.0",
    "jupyterlab-git>=0.41.0",
]

[tool.black]
line-length = 88
target-version = ['py39', 'py310', 'py311']
include = '\.pyi?$'
extend-exclude = '''
/(
  # directories
  \.eggs
  | \.git
  | \.hg
  | \.mypy_cache
  | \.tox
  | \.venv
  | build
  | dist
)/
'''

[tool.mypy]
python_version = "3.11"
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true
```

### Environment Configuration

```bash
# Create .env file for environment variables
cat > .env << 'EOF'
# Python environment
PYTHONPATH=./src
PYTHONDONTWRITEBYTECODE=1
PYTHONUNBUFFERED=1

# Jupyter configuration
JUPYTER_ENABLE_LAB=yes
JUPYTER_TOKEN=

# ML Framework settings
CUDA_VISIBLE_DEVICES=0
TF_CPP_MIN_LOG_LEVEL=2
PYTORCH_CUDA_ALLOC_CONF=max_split_size_mb:512

# Data directories
DATA_DIR=./data
MODEL_DIR=./models
NOTEBOOK_DIR=./notebooks

# API keys (add your actual keys)
OPENAI_API_KEY=your_openai_key_here
HUGGINGFACE_TOKEN=your_hf_token_here
WANDB_API_KEY=your_wandb_key_here
EOF

# Add .env to .gitignore
echo ".env" >> .gitignore
```

## Sample Data Science Projects

### Project 1: Data Analysis Pipeline

```python
# src/data_pipeline.py
import pandas as pd
import numpy as np
from pathlib import Path
import logging

# Setup logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class DataPipeline:
    def __init__(self, data_dir: Path):
        self.data_dir = Path(data_dir)
        self.raw_dir = self.data_dir / "raw"
        self.processed_dir = self.data_dir / "processed"

    def load_raw_data(self, filename: str) -> pd.DataFrame:
        """Load raw data from CSV file."""
        filepath = self.raw_dir / filename
        logger.info(f"Loading data from {filepath}")
        return pd.read_csv(filepath)

    def clean_data(self, df: pd.DataFrame) -> pd.DataFrame:
        """Clean and preprocess data."""
        logger.info("Cleaning data...")

        # Remove duplicates
        df = df.drop_duplicates()

        # Handle missing values
        numeric_columns = df.select_dtypes(include=[np.number]).columns
        df[numeric_columns] = df[numeric_columns].fillna(df[numeric_columns].median())

        # Handle categorical missing values
        categorical_columns = df.select_dtypes(include=['object']).columns
        df[categorical_columns] = df[categorical_columns].fillna('Unknown')

        return df

    def save_processed_data(self, df: pd.DataFrame, filename: str):
        """Save processed data."""
        filepath = self.processed_dir / filename
        logger.info(f"Saving processed data to {filepath}")
        df.to_csv(filepath, index=False)

# Usage example
if __name__ == "__main__":
    pipeline = DataPipeline("./data")

    # Load, clean, and save data
    raw_data = pipeline.load_raw_data("sample_data.csv")
    clean_data = pipeline.clean_data(raw_data)
    pipeline.save_processed_data(clean_data, "cleaned_data.csv")
```

### Project 2: Machine Learning Model Training

```python
# src/ml_model.py
import pandas as pd
import numpy as np
from sklearn.model_selection import train_test_split, GridSearchCV
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import classification_report, confusion_matrix
from sklearn.preprocessing import StandardScaler, LabelEncoder
import joblib
from pathlib import Path
import logging

logger = logging.getLogger(__name__)

class MLModelTrainer:
    def __init__(self, model_dir: Path):
        self.model_dir = Path(model_dir)
        self.model = None
        self.scaler = StandardScaler()
        self.label_encoder = LabelEncoder()

    def prepare_features(self, df: pd.DataFrame, target_column: str):
        """Prepare features for training."""
        logger.info("Preparing features...")

        # Separate features and target
        X = df.drop(columns=[target_column])
        y = df[target_column]

        # Handle categorical variables
        categorical_columns = X.select_dtypes(include=['object']).columns
        for col in categorical_columns:
            le = LabelEncoder()
            X[col] = le.fit_transform(X[col].astype(str))

        # Scale features
        X_scaled = self.scaler.fit_transform(X)

        # Encode target if categorical
        if y.dtype == 'object':
            y_encoded = self.label_encoder.fit_transform(y)
        else:
            y_encoded = y

        return X_scaled, y_encoded

    def train_model(self, X, y, test_size=0.2):
        """Train the machine learning model."""
        logger.info("Training model...")

        # Split data
        X_train, X_test, y_train, y_test = train_test_split(
            X, y, test_size=test_size, random_state=42, stratify=y
        )

        # Define model and parameters for grid search
        rf = RandomForestClassifier(random_state=42)
        param_grid = {
            'n_estimators': [100, 200, 300],
            'max_depth': [10, 20, None],
            'min_samples_split': [2, 5, 10]
        }

        # Grid search
        grid_search = GridSearchCV(
            rf, param_grid, cv=5, scoring='accuracy', n_jobs=-1
        )
        grid_search.fit(X_train, y_train)

        self.model = grid_search.best_estimator_

        # Evaluate model
        y_pred = self.model.predict(X_test)

        logger.info("Model Performance:")
        logger.info(f"Best parameters: {grid_search.best_params_}")
        logger.info(f"Best cross-validation score: {grid_search.best_score_:.4f}")
        logger.info("\nClassification Report:")
        logger.info(classification_report(y_test, y_pred))

        return X_test, y_test, y_pred

    def save_model(self, model_name: str):
        """Save trained model and preprocessors."""
        model_path = self.model_dir / f"{model_name}.joblib"
        scaler_path = self.model_dir / f"{model_name}_scaler.joblib"
        encoder_path = self.model_dir / f"{model_name}_encoder.joblib"

        joblib.dump(self.model, model_path)
        joblib.dump(self.scaler, scaler_path)
        joblib.dump(self.label_encoder, encoder_path)

        logger.info(f"Model saved to {model_path}")

# Usage example
if __name__ == "__main__":
    trainer = MLModelTrainer("./models")

    # Load processed data
    df = pd.read_csv("./data/processed/cleaned_data.csv")

    # Prepare features and train model
    X, y = trainer.prepare_features(df, "target_column")
    X_test, y_test, y_pred = trainer.train_model(X, y)

    # Save model
    trainer.save_model("random_forest_model")
```

### Project 3: Deep Learning with PyTorch

```python
# src/deep_learning.py
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader, Dataset
import pandas as pd
import numpy as np
from sklearn.preprocessing import StandardScaler
from sklearn.model_selection import train_test_split
import logging

logger = logging.getLogger(__name__)

class TabularDataset(Dataset):
    def __init__(self, X, y):
        self.X = torch.FloatTensor(X)
        self.y = torch.LongTensor(y)

    def __len__(self):
        return len(self.X)

    def __getitem__(self, idx):
        return self.X[idx], self.y[idx]

class NeuralNetwork(nn.Module):
    def __init__(self, input_size, hidden_sizes, num_classes, dropout_rate=0.2):
        super(NeuralNetwork, self).__init__()

        layers = []
        prev_size = input_size

        for hidden_size in hidden_sizes:
            layers.extend([
                nn.Linear(prev_size, hidden_size),
                nn.ReLU(),
                nn.BatchNorm1d(hidden_size),
                nn.Dropout(dropout_rate)
            ])
            prev_size = hidden_size

        layers.append(nn.Linear(prev_size, num_classes))
        self.network = nn.Sequential(*layers)

    def forward(self, x):
        return self.network(x)

class DeepLearningTrainer:
    def __init__(self, model_dir: str):
        self.model_dir = model_dir
        self.device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
        logger.info(f"Using device: {self.device}")

    def prepare_data(self, df: pd.DataFrame, target_column: str, test_size=0.2):
        """Prepare data for deep learning."""
        X = df.drop(columns=[target_column]).values
        y = df[target_column].values

        # Scale features
        scaler = StandardScaler()
        X_scaled = scaler.fit_transform(X)

        # Split data
        X_train, X_test, y_train, y_test = train_test_split(
            X_scaled, y, test_size=test_size, random_state=42
        )

        # Create datasets
        train_dataset = TabularDataset(X_train, y_train)
        test_dataset = TabularDataset(X_test, y_test)

        return train_dataset, test_dataset, scaler

    def train_model(self, train_dataset, test_dataset, input_size, num_classes,
                   epochs=100, batch_size=32, learning_rate=0.001):
        """Train the neural network."""

        # Create data loaders
        train_loader = DataLoader(train_dataset, batch_size=batch_size, shuffle=True)
        test_loader = DataLoader(test_dataset, batch_size=batch_size, shuffle=False)

        # Initialize model
        model = NeuralNetwork(
            input_size=input_size,
            hidden_sizes=[128, 64, 32],
            num_classes=num_classes
        ).to(self.device)

        # Loss and optimizer
        criterion = nn.CrossEntropyLoss()
        optimizer = optim.Adam(model.parameters(), lr=learning_rate)
        scheduler = optim.lr_scheduler.StepLR(optimizer, step_size=30, gamma=0.1)

        # Training loop
        for epoch in range(epochs):
            model.train()
            train_loss = 0.0

            for batch_X, batch_y in train_loader:
                batch_X, batch_y = batch_X.to(self.device), batch_y.to(self.device)

                optimizer.zero_grad()
                outputs = model(batch_X)
                loss = criterion(outputs, batch_y)
                loss.backward()
                optimizer.step()

                train_loss += loss.item()

            scheduler.step()

            # Validation
            if epoch % 10 == 0:
                model.eval()
                test_loss = 0.0
                correct = 0
                total = 0

                with torch.no_grad():
                    for batch_X, batch_y in test_loader:
                        batch_X, batch_y = batch_X.to(self.device), batch_y.to(self.device)
                        outputs = model(batch_X)
                        loss = criterion(outputs, batch_y)
                        test_loss += loss.item()

                        _, predicted = torch.max(outputs.data, 1)
                        total += batch_y.size(0)
                        correct += (predicted == batch_y).sum().item()

                accuracy = 100 * correct / total
                logger.info(f'Epoch [{epoch+1}/{epochs}], '
                          f'Train Loss: {train_loss/len(train_loader):.4f}, '
                          f'Test Loss: {test_loss/len(test_loader):.4f}, '
                          f'Accuracy: {accuracy:.2f}%')

        return model

    def save_model(self, model, model_name: str):
        """Save the trained model."""
        model_path = f"{self.model_dir}/{model_name}.pth"
        torch.save(model.state_dict(), model_path)
        logger.info(f"Model saved to {model_path}")

# Usage example
if __name__ == "__main__":
    trainer = DeepLearningTrainer("./models")

    # Load data
    df = pd.read_csv("./data/processed/cleaned_data.csv")

    # Prepare data
    train_dataset, test_dataset, scaler = trainer.prepare_data(df, "target_column")

    # Train model
    input_size = df.shape[1] - 1  # excluding target column
    num_classes = df["target_column"].nunique()

    model = trainer.train_model(
        train_dataset, test_dataset, input_size, num_classes
    )

    # Save model
    trainer.save_model(model, "neural_network_model")
```

## Development Workflow and Best Practices

### Pre-commit Hooks Setup

```bash
# Install pre-commit
uv add --dev pre-commit

# Create pre-commit configuration
cat > .pre-commit-config.yaml << 'EOF'
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.4.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-added-large-files
      - id: check-merge-conflict

  - repo: https://github.com/psf/black
    rev: 23.7.0
    hooks:
      - id: black
        language_version: python3.11

  - repo: https://github.com/pycqa/flake8
    rev: 6.0.0
    hooks:
      - id: flake8
        args: [--max-line-length=88, --extend-ignore=E203,W503]

  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.5.1
    hooks:
      - id: mypy
        additional_dependencies: [types-requests]
EOF

# Install pre-commit hooks
uv run pre-commit install
```

### Testing Framework

```python
# tests/test_data_pipeline.py
import pytest
import pandas as pd
import numpy as np
from pathlib import Path
import tempfile
import sys
sys.path.append('./src')

from data_pipeline import DataPipeline

class TestDataPipeline:
    @pytest.fixture
    def sample_data(self):
        """Create sample data for testing."""
        return pd.DataFrame({
            'feature1': [1, 2, np.nan, 4, 5],
            'feature2': ['A', 'B', 'C', 'D', 'A'],
            'feature3': [10.5, 20.3, 30.1, np.nan, 50.7],
            'target': [0, 1, 0, 1, 0]
        })

    @pytest.fixture
    def temp_pipeline(self):
        """Create temporary pipeline for testing."""
        with tempfile.TemporaryDirectory() as temp_dir:
            temp_path = Path(temp_dir)
            (temp_path / "raw").mkdir()
            (temp_path / "processed").mkdir()
            yield DataPipeline(temp_path)

    def test_clean_data(self, temp_pipeline, sample_data):
        """Test data cleaning functionality."""
        cleaned_data = temp_pipeline.clean_data(sample_data)

        # Check that missing values are handled
        assert not cleaned_data.isnull().any().any()

        # Check that duplicates are removed
        assert len(cleaned_data) <= len(sample_data)

    def test_save_processed_data(self, temp_pipeline, sample_data):
        """Test saving processed data."""
        filename = "test_data.csv"
        temp_pipeline.save_processed_data(sample_data, filename)

        # Check that file was created
        filepath = temp_pipeline.processed_dir / filename
        assert filepath.exists()

        # Check that data can be loaded back
        loaded_data = pd.read_csv(filepath)
        pd.testing.assert_frame_equal(sample_data, loaded_data)

# Run tests
# uv run pytest tests/ -v
```

### Makefile for Common Tasks

```makefile
# Makefile
.PHONY: install test lint format clean jupyter notebook

# Install dependencies
install:
	uv sync

# Run tests
test:
	uv run pytest tests/ -v --cov=src

# Lint code
lint:
	uv run flake8 src/ tests/
	uv run mypy src/

# Format code
format:
	uv run black src/ tests/
	uv run isort src/ tests/

# Clean cache and temporary files
clean:
	find . -type f -name "*.pyc" -delete
	find . -type d -name "__pycache__" -delete
	find . -type d -name ".pytest_cache" -exec rm -rf {} +
	find . -type d -name ".mypy_cache" -exec rm -rf {} +

# Start Jupyter Lab
jupyter:
	uv run jupyter lab --ip=0.0.0.0 --port=8888 --no-browser

# Start Jupyter Notebook
notebook:
	uv run jupyter notebook --ip=0.0.0.0 --port=8888 --no-browser

# Install development dependencies
dev-install:
	uv sync --all-extras

# Update dependencies
update:
	uv lock --upgrade
	uv sync

# Build documentation
docs:
	uv run sphinx-build -b html docs/ docs/_build/

# Run security check
security:
	uv run bandit -r src/

# Profile code
profile:
	uv run python -m cProfile -o profile.stats src/main.py
	uv run python -c "import pstats; pstats.Stats('profile.stats').sort_stats('cumulative').print_stats(20)"
```

## Advanced Configurations

### GPU Configuration for Deep Learning

```bash
# Check GPU availability
uv run python -c "
import torch
print(f'CUDA available: {torch.cuda.is_available()}')
print(f'CUDA version: {torch.version.cuda}')
print(f'GPU count: {torch.cuda.device_count()}')
if torch.cuda.is_available():
    print(f'GPU name: {torch.cuda.get_device_name(0)}')
"

# TensorFlow GPU check
uv run python -c "
import tensorflow as tf
print(f'TensorFlow version: {tf.__version__}')
print(f'GPU devices: {tf.config.list_physical_devices(\"GPU\")}')
"
```

### Memory Optimization

```python
# src/memory_utils.py
import psutil
import gc
import torch
import logging

logger = logging.getLogger(__name__)

class MemoryManager:
    @staticmethod
    def get_memory_usage():
        """Get current memory usage."""
        process = psutil.Process()
        memory_info = process.memory_info()
        return {
            'rss': memory_info.rss / 1024 / 1024,  # MB
            'vms': memory_info.vms / 1024 / 1024,  # MB
            'percent': process.memory_percent()
        }

    @staticmethod
    def clear_memory():
        """Clear Python and GPU memory."""
        gc.collect()
        if torch.cuda.is_available():
            torch.cuda.empty_cache()
        logger.info("Memory cleared")

    @staticmethod
    def log_memory_usage(stage: str):
        """Log memory usage at different stages."""
        memory = MemoryManager.get_memory_usage()
        logger.info(f"{stage} - Memory usage: {memory['rss']:.1f}MB ({memory['percent']:.1f}%)")

        if torch.cuda.is_available():
            gpu_memory = torch.cuda.memory_allocated() / 1024 / 1024
            logger.info(f"{stage} - GPU memory: {gpu_memory:.1f}MB")

# Usage in training scripts
memory_manager = MemoryManager()
memory_manager.log_memory_usage("Before training")
# ... training code ...
memory_manager.clear_memory()
memory_manager.log_memory_usage("After training")
```

### Distributed Computing Setup

```python
# src/distributed_utils.py
import dask
from dask.distributed import Client, as_completed
import pandas as pd
import numpy as np

class DistributedProcessor:
    def __init__(self, n_workers=4):
        self.client = Client(n_workers=n_workers, threads_per_worker=2)
        print(f"Dask dashboard: {self.client.dashboard_link}")

    def parallel_processing(self, data_chunks, processing_func):
        """Process data chunks in parallel."""
        futures = []
        for chunk in data_chunks:
            future = self.client.submit(processing_func, chunk)
            futures.append(future)

        results = []
        for future in as_completed(futures):
            result = future.result()
            results.append(result)

        return results

    def close(self):
        """Close the Dask client."""
        self.client.close()

# Example usage
def process_chunk(df_chunk):
    """Example processing function."""
    return df_chunk.groupby('category').agg({
        'value': ['mean', 'std', 'count']
    }).reset_index()

# Usage
processor = DistributedProcessor(n_workers=8)
# ... use processor for parallel computations ...
processor.close()
```

## Monitoring and Profiling

### Performance Monitoring

```python
# src/monitoring.py
import time
import functools
import cProfile
import pstats
from memory_profiler import profile
import logging

logger = logging.getLogger(__name__)

def timing_decorator(func):
    """Decorator to measure function execution time."""
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start_time = time.time()
        result = func(*args, **kwargs)
        end_time = time.time()
        logger.info(f"{func.__name__} took {end_time - start_time:.4f} seconds")
        return result
    return wrapper

def profile_function(func):
    """Decorator to profile function performance."""
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        profiler = cProfile.Profile()
        profiler.enable()
        result = func(*args, **kwargs)
        profiler.disable()

        stats = pstats.Stats(profiler)
        stats.sort_stats('cumulative')
        stats.print_stats(20)

        return result
    return wrapper

# Usage examples
@timing_decorator
@profile
def expensive_computation(data):
    """Example of monitored function."""
    return data.apply(lambda x: x ** 2).sum()
```

### Experiment Tracking

```python
# src/experiment_tracking.py
import mlflow
import mlflow.sklearn
import mlflow.pytorch
from pathlib import Path
import json

class ExperimentTracker:
    def __init__(self, experiment_name: str, tracking_uri: str = None):
        if tracking_uri:
            mlflow.set_tracking_uri(tracking_uri)

        mlflow.set_experiment(experiment_name)
        self.run = None

    def start_run(self, run_name: str = None):
        """Start MLflow run."""
        self.run = mlflow.start_run(run_name=run_name)
        return self.run

    def log_params(self, params: dict):
        """Log parameters."""
        mlflow.log_params(params)

    def log_metrics(self, metrics: dict, step: int = None):
        """Log metrics."""
        for key, value in metrics.items():
            mlflow.log_metric(key, value, step=step)

    def log_model(self, model, model_name: str, framework: str = "sklearn"):
        """Log model."""
        if framework == "sklearn":
            mlflow.sklearn.log_model(model, model_name)
        elif framework == "pytorch":
            mlflow.pytorch.log_model(model, model_name)

    def log_artifacts(self, artifact_path: str):
        """Log artifacts."""
        mlflow.log_artifacts(artifact_path)

    def end_run(self):
        """End MLflow run."""
        if self.run:
            mlflow.end_run()

# Usage example
tracker = ExperimentTracker("data-science-experiments")
with tracker.start_run("model-training-v1"):
    tracker.log_params({"learning_rate": 0.01, "epochs": 100})
    # ... training code ...
    tracker.log_metrics({"accuracy": 0.95, "loss": 0.05})
    tracker.log_model(trained_model, "random_forest_v1")
```

## Deployment and Production

### Model Serving with FastAPI

```python
# src/model_server.py
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
import joblib
import numpy as np
import pandas as pd
from pathlib import Path
import logging

logger = logging.getLogger(__name__)

app = FastAPI(title="ML Model API", version="1.0.0")

# Load model at startup
MODEL_PATH = Path("./models")
model = joblib.load(MODEL_PATH / "random_forest_model.joblib")
scaler = joblib.load(MODEL_PATH / "random_forest_model_scaler.joblib")

class PredictionRequest(BaseModel):
    features: list[float]

class PredictionResponse(BaseModel):
    prediction: int
    probability: list[float]

@app.post("/predict", response_model=PredictionResponse)
async def predict(request: PredictionRequest):
    try:
        # Prepare features
        features = np.array(request.features).reshape(1, -1)
        features_scaled = scaler.transform(features)

        # Make prediction
        prediction = model.predict(features_scaled)[0]
        probabilities = model.predict_proba(features_scaled)[0].tolist()

        return PredictionResponse(
            prediction=int(prediction),
            probability=probabilities
        )

    except Exception as e:
        logger.error(f"Prediction error: {str(e)}")
        raise HTTPException(status_code=500, detail=str(e))

@app.get("/health")
async def health_check():
    return {"status": "healthy"}

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

### Docker Configuration

```dockerfile
# Dockerfile
FROM python:3.11-slim

# Install system dependencies
RUN apt-get update && apt-get install -y \
    build-essential \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Install UV
RUN curl -LsSf https://astral.sh/uv/install.sh | sh
ENV PATH="/root/.local/bin:$PATH"

# Set working directory
WORKDIR /app

# Copy project files
COPY pyproject.toml uv.lock ./
COPY src/ ./src/
COPY models/ ./models/

# Install dependencies
RUN uv sync --frozen

# Expose port
EXPOSE 8000

# Run the application
CMD ["uv", "run", "python", "src/model_server.py"]
```

## Conclusion

Setting up a modern Python data science and AI development environment with UV provides significant advantages in terms of speed, reliability, and maintainability. This comprehensive setup gives you:

**Key Benefits:**
- **Lightning-fast package management** with UV's superior performance
- **Reproducible environments** with lockfiles and version pinning
- **Professional project structure** following data science best practices
- **Complete ML/AI toolkit** with PyTorch, TensorFlow, and Hugging Face
- **Advanced development workflow** with testing, linting, and monitoring
- **Production-ready deployment** with FastAPI and Docker

**What You've Accomplished:**
- Modern Python environment with multiple version support
- Complete data science stack with all essential packages
- Machine learning frameworks properly configured
- Professional development workflow with testing and CI/CD
- Monitoring and experiment tracking capabilities
- Production deployment setup

This environment setup scales from individual research projects to enterprise-level AI applications. The combination of UV's speed, comprehensive package management, and modern development practices creates an ideal foundation for data science and AI development.

Whether you're conducting exploratory data analysis, training complex deep learning models, or deploying AI applications to production, this setup provides the tools and structure you need to be productive and successful in your data science journey.