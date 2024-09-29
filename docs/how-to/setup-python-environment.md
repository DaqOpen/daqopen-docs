# Setup Python Environment

It is good practice today to use a virtual environment for python projects. For working with the examples and tutorials I highly recommend to setup an appropriate environment. With the most tools this is a matter of minutes.



## Linux

1. Install the system package for creating the virtual environment (we will use venv here). Execute the following command in the terminal:
   **Debian/Ubuntu:** `sudo apt install python3-venv` 
   **Fedora:** `sudo dnf install python3-venv`

2. Create a project folder and navigate inside:

   ```bash
   mkdir daqopen-experiments
   cd daqopen-experiments
   ```

3. Create the Virtual Environment (located in the folder .venv) and activate it:

   ```bash
   python3 -m venv .venv
   source .venv/bin/activate
   ```

4. Install the packages required for the most examples:

   ```bash
   pip install daqopen-lib
   pip install matplotlib
   pip install PyQt6
   ```

5. Open an IDE or code editor of your preference. In the tutorials I use most of the time VS Code, which can be downloaded here: https://code.visualstudio.com/Download



## Windows

Use this Tutorial to Setup VS Code and Python: