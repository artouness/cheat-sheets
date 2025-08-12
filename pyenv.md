Using `pyenv` along with `pyenv-virtualenv` is an excellent way to manage multiple Python versions and isolated environments per project.

Here’s a **full guide** to installing, creating, activating, deactivating, and deleting virtual environments using `pyenv` and `pyenv-virtualenv`.

### 🛠️ **1. Install `pyenv` and `pyenv-virtualenv`**

If you haven’t installed these already:

```bash
# Install pyenv
curl https://pyenv.run | bash
```

Then add to your `~/.bashrc`, `~/.zshrc`, or `~/.profile`:

```bash
export PATH="$HOME/.pyenv/bin:$PATH"
eval "$(pyenv init --path)"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
```

Then restart your shell:
```bash
exec "$SHELL"
```

Or use:
```bash
source ~/.bashrc  # or ~/.zshrc
```

Install `pyenv-virtualenv` if not included:
```bash
git clone https://github.com/pyenv/pyenv-virtualenv.git "$(pyenv root)/plugins/pyenv-virtualenv"
```

### 📦 **2. Install a Python Version**

```bash
pyenv install 3.10.13
```

### 🧪 **3. Create a Virtual Environment**

```bash
pyenv virtualenv 3.10.13 myproject-env
```

You can name it whatever you want (`myproject-env` here).

### 🚀 **4. Activate a Virtual Environment**

You have two ways:

#### ✅ Manual activation:
```bash
pyenv activate myproject-env
```

#### ✅ Auto-activate in project folder:
In your project directory:

```bash
pyenv local myproject-env
```

Now whenever you `cd` into that folder, it will auto-activate the environment.

### 🛑 **5. Deactivate a Virtual Environment**

```bash
pyenv deactivate
```

### 🗑️ **6. Delete a Virtual Environment**

```bash
pyenv uninstall myproject-env
```

### 🧹 Optional: List Versions and Envs

```bash
pyenv versions
```

Shows all installed Python versions and virtualenvs.
