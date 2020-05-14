# Amortization-Calculator-








import numpy as np
import pandas as pd
from datetime import date
import matplotlib
import matplotlib.pyplot as plt


#%matplotlib tk 
matplotlib.style.use('ggplot')

def Amortization_table(interest_rate, years, payments_year, principal, addl_principal=0, start_date=date.today()):
    #This will be the working function for calculating the Amortization table and schedule for loans.
    
     #The arguments for creating an amortization function include:
        #interest_rate: This is the annual interest rate for a loan.
        #years: Number of years for the loan.
        #payments_year: Number of payments to be made in a year.
        #principal: The amount borrowed to pay, which interest is paid.
        #addl_principal (optional): Additional payments to be made. Assume 0 if nothing provided. 
        #This must be a value less then 0, the function will make a positive value to negative.
        #start_date (optional): This is the start date of the payments. This will start on the first day of next month if no date is provided.

    #What will be returned in this function:
        #schedule: Amortization schedule from the 'pandas' dataframe
        #summary: Pandas dataframe that will summerize the payoff information
    
    
    
    
    
    
    
    #This will make the additional payments negative in the function. 
    if addl_principal > 0:
        addl_principal = -addl_principal
    
    #This will make create an index for payments
    rng = pd.date_range(start_date, periods=years * payments_year, freq='MS')
    rng.name = "Payment_Date"
    
    #This creats up the Amortization schedule as a DataFrame
    df = pd.DataFrame(index=rng,columns=['Payment', 'Principal', 'Interest', 'Addl_Principal', 'Current_Balance'], dtype='float')
    
    #Add index by period (start at 1 not 0)
    df.reset_index(inplace=True)
    df.index += 1
    df.index.name = "Period"
    
    #This calculates the payment, principal and interests amounts using built in "numpy" functions
    per_payment = np.pmt(interest_rate/payments_year, years*payments_year, principal)
    df["Payment"] = per_payment
    df["Principal"] = np.ppmt(interest_rate/payments_year, df.index, years*payments_year, principal)
    df["Interest"] = np.ipmt(interest_rate/payments_year, df.index, years*payments_year, principal)
        
    #Round the values
    df = df.round(2) 
    #Add in the additional principal payments
    df["Addl_Principal"] = addl_principal
    
    #Cumulative Principal Payments and makes it stop before it gets larger than the original principal
    df["Cumulative_Principal"] = (df["Principal"] + df["Addl_Principal"]).cumsum()
    df["Cumulative_Principal"] = df["Cumulative_Principal"].clip(lower=-principal)
    
    #Calculate the current balance for each period
    df["Current_Balance"] = principal + df["Cumulative_Principal"]
    
    #Determine the last payment date for the loan
    try:
        last_payment = df.query("Current_Balance <= 0")["Current_Balance"].idxmax(axis=1, skipna=True)
    except ValueError:
        last_payment = df.last_valid_index()
    
    last_payment_date = "{:%m-%d-%Y}".format(df.loc[last_payment, "Payment_Date"])
   










    """Here is the Additional Principle Payments Section"""
    #Additional Principle Payments 
    #Shorten the data frame if we have additional principle payments:
    if addl_principal != 0:
                
        # Remove the extra payment periods
        # df.loc gets a group of rows and columns by labels
        df = df.loc[0:last_payment].copy()
        # Calculate the principal for the last row
        df.loc[last_payment, "Principal"] = -(df.loc[last_payment-1, "Current_Balance"])
        # Calculate the total payment for the last row
        df.loc[last_payment, "Payment"] = df.loc[last_payment, ["Principal", "Interest"]].sum()
        # Zero out the additional principal
        df.loc[last_payment, "Addl_Principal"] = 0
        
    #Get the payment info into a DataFrame in column order
    payment_info = (df[["Payment", "Principal", "Addl_Principal", "Interest"]] .sum().to_frame().T)
       
    #create a Format the Date DataFrame
    payment_details = pd.DataFrame.from_dict(dict([('payoff_date', [last_payment_date]), ('Interest Rate', [interest_rate]),  ('Number of years', [years])]))
   
    # Add a column showing how much we pay each period.
    # Combine addl principal with principal for total payment
    payment_details["Period_Payment"] = round(per_payment, 2) + addl_principal
    
    payment_summary = pd.concat([payment_details, payment_info], axis=1)
    return df, payment_summary
    # Still needs tweek on monthly/yearly payments 







"""Here is the visualization for graph and tables"""

    #here is examples of running the function 
schedule1, stats1 = Amortization_table(0.05, 30, 12, 140000, addl_principal=0)
schedule2, stats2 = Amortization_table(0.05, 30, 12, 140000, addl_principal=-100)
schedule3, stats3 = Amortization_table(0.04, 15, 12, 140000, addl_principal=0)

    #This will allow for an example of the table of schedule for the loan payments
    #and also stats about the table.
schedule1.head()
schedule2.head()
schedule3.head()
schedule1.tail()
schedule2.tail()
schedule3.tail()
stats1
stats2
stats3
    #This is all individual examples of the amortization tables.
    #This joins the stats together and presents the final scenarios on loan payment tables
pd.concat([stats1, stats2, stats3], ignore_index=True)








"""This is the graph options"""

    #This will plot the data on a graph. 
schedule1.plot(x='Payment_Date', y='Current_Balance', title="Schedule Timeline 1");
schedule2.plot(x='Payment_Date', y='Current_Balance', title="Scenario Timeline 2");
schedule3.plot(x='Payment_Date', y='Current_Balance', title="Scenario Timeline 3");

    #
fig, ax = plt.subplots(1, 1)
schedule1.plot(x='Payment_Date', y='Current_Balance', label="Schedule 1", ax=ax)
schedule2.plot(x='Payment_Date', y='Current_Balance', label="Schedule 2", ax=ax)
schedule3.plot(x='Payment_Date', y='Current_Balance', label="Schedule 3", ax=ax)
plt.title("Pay Off Schedule");





