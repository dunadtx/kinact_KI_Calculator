# kinact and KI Calculator

## Citations
This code is free to use but please cite "Mader & Keillor, ACS Med. Chem. Lett. **2024**, *15*, xxxx" and the GitHub link when using it. <br>

The Python code was developed and is maintained by Bethany Atkinson at [Dunad Therapeutics](https://www.dunadtx.com/).

## Overview

The code implements the method described in Mader & Keillor, ACS Med. Chem. Lett. **2024**, *15*, xxxx for the calculation of kinact and KI. <br>

Through the simulation of the reaction the predicted product concentration is calculated at the end of each IC50 experiment. The predicted product concentration is scaled to give a % signal. The kinact and KI values are determined by using scipy least_squares to optimise the predicted signal to the experimentally observed signal. 

Compared to the Excel spreadsheet (referenced below) which can also be used to determine the kinact and KI values, the Python code uses [scipy least_squares](https://docs.scipy.org/doc/scipy/reference/generated/scipy.optimize.least_squares.html#r20fc1df64af7-stir) rather than Excel's Solver to optimise the kinact and KI values. The Excel Solver uses the GRG (Generalised Reduced Gradient) non-linear algorithm. scipy least_squares uses the Trust Region Reflective algorithm for the optimisation problem. The two give very similar results.

Additionally, the Python code calculates the response coefficient which has to be set manually when using the Excel spreadsheet to calculate the kinact and KI values. The response coefficient is calculated by making the predicted % signal equal to 100 when inhibitor concentration is 0 µM using the input/initial kinact and KI estimates. Once the kinact and KI values have been calculated the predicted signal with 0 µM should still be approximately equal to 100%.

## Excel Spreadsheet

An example of the Excel spreadsheet which can also be used to calculate the kinact and KI values using the same method can be found in the 'Excel' folder. 

## Dependencies
- numpy
- scipy
- matplotlib
- pandas

## Geting Started
Clone the git repository.

To use the kinact and KI calculator in a Jupyter notebook, navigate to the working directory with the downloaded code and run the following command in the Jupyter notebook: <br>

`from kinact_KI_calculator import KineticParameterCalculator`

## Usage
kinact_KI_calculator_example_notebook.ipynb is an example of the code being used to calculate the kinact and KI values in a Jupyter notebook. <br>

The file 'Sample data sheet.csv' shows an example input file. Read in the IC50 curve data and create a numpy array. The data for input to the kinact/KI calculator should be a numpy array with 4 columns: Pre-incubation time (min), Incubation time (min), [Inhibitor] (µM), Signal (%). <br>

To run the code call an instance of the class: <br>

`>>> calculated_kinact_KI = KineticParameterCalculator(array, AddSub, EnzConc, kcat, Km, PreIncVol, IncVol)` <br>
<br>
The inputs are:
- a numpy array with 4 columns: Pre-incubation time (min), Incubation time (min), [Inhibitor] (µM), Signal (%)
- AddSub: the substrate concentration (µM) used in the experiment. 
- EnzConc: the enzyme concentration (µM) used in producing the IC50 curves.
- kcat
- Km
- PreIncVol: the pre-incubation volume (µL)
- IncVol: the incubation volume (µL) after substrate addition.

Optionally the starting kinact and KI values in min-1 and µM respectively can be added as inputs. These are initial guesses which go into the least_squares optimisation algorithm. If no starting kinact or KI value is added a default of 1 for each value will be used. <br>

The least_squares minimisation used to determine the kinact and KI values may find a local rather than a global minimum. Changing the input kinact and KI values may lead to different values that allow local convergence. 

Creating the instance automatically calculates the kinact and KI values which can then be printed: <br>

`>>> print("kinact = " + str(test._kinact))` <br>
`>>>print("KI = " + str (test._KI))` <br>

Running the code also adds a new column to the numpy array of the predicted signal (calculated using the calculated kinact and KI values). <br>

The experimentally observed signal compared to the predicted signal can be plotted using the following code: <br>

    ```
    df = pd.DataFrame(test.array, columns =['Pre-incubation time (min)', 'Incubation time (min)', '[Inhibitor] (µM)', 'Signal', 'Predicted signal'])
    for i in df.groupby('Pre-incubation time (min)'):
        plt.title('Pre-incubation time: {} mins '.format(str(i[0])))
        plt.scatter(i[1]['[Inhibitor] (µM)'], i[1]['Signal'], label='Signal') #plot signal
        plt.plot(i[1]['[Inhibitor] (µM)'], i[1]['Predicted signal'], label='Pred Signal', c='r', marker = 's') #plot predicted signal
        plt.xscale('log') #set x-axis to log scale 
        plt.xlabel("Inhibitor Concentration (uM)")
        plt.ylabel("Signal")
        plt.legend()
        plt.show()
    ```

