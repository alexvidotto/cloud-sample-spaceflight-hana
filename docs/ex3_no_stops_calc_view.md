# Exercise 3: Calculation View Showing Direct Flights

## Before you Start...

In order to start this exercise, you must first have compiled and deployed your Spaceflight data model to HANA.  See pre-requisite exercises [0.3](https://github.com/SAP/cloud-sample-spaceflight-hana/blob/master/docs/ex0.4.md) for details.

## Exercise Steps

1. Right click on the `db\src` folder and create a new calculation view by selecting New -> Calculation View

    ![New Calculation View](./img/Ex3_New_Calc_View.png)

1. Give the new calculation view the name `route0stop`, ensure that the Data Category is `DEFAULT` and press Create

    ![Calculation View Name](./img/Ex3_Route0Stops.png)

1. Now we need to add a projection.  If it is not already expanded, expand the sidebar menu on the calculation view editor by clicking the `>>` icon.

    Select the "Projection" tool then click somewhere underneath the "Projection" icon to insert a new projection

    ![Calculation View Name](./img/Ex3_New_Projection.png)

1. With the new projection selected, add a new data source by clicking on the `+` icon

    ![New Data Source](./img/Ex3_New_Data_Src.png)


1. In the pop-up window, enter "earthroutes" and in the table below you should see the name of the generated table.  Select `TECHED_FLIGHT_TRIP_EARTHROUTES` and click Finish

    ![Choose Table Name](./img/Ex3_Choose_Data_Src.png)

1. Once you have added the data source to the projection, you now see that the table name has been added to the `Projection_1` icon.

    ![Calculation View](./img/Ex3_Calc_View.png)

    We now need to open `Projection_1`'s detail pane.  To do this, select `Projection_1` and either click on the "Expand Detail Pane" icon ![Expand Detail Pane](./img/Icon_Detail_Pane.png),  or grab the handle on the right side of the screen and drag the detail pane out towards the left.
    
    ![Detail Pane](./img/Ex3_Detail_Pane.png)

1. We must now decide which columns in the underlying database table will be "projected" though to the calculation view.  In this case, we will take the simple option and select everything! ![Icon Smiley](./img/Icon_Smiley.png)

    Click on the table name and drag the entire table to the "Output Columns" area on the right.

    ![Data Mapping](./img/Ex3_Data_Mapping_Before.png)

1. Now, every column in the `TECHED_FLIGHT_TRIP_EARTHROUTES` table will be projected through to the calculation view.

    ![Data Mapping](./img/Ex3_Data_Mapping_After.png)

1. If we stopped at this point, there would be little point in having a projection, since all we have done so far is directly expose the entire underlying database table.

    So in order to allow the table data to be filtered, we will now add two input parameters called `airportFrom` and `airportTo`.

    | Name | Is Mandatory | Data Type | Length |
    |---|:-:|---|:-:|
    | `airportFrom` | ![Tick](./img/Icon_Tick.png) | `NVARCHAR` | 3
    | `airportTo` | ![Tick](./img/Icon_Tick.png) | `NVARCHAR` | 3
    
    For each of the input parameters shown in the table above, apply the listed properties by following these steps:

    1. From the Projection screen, select the "Parameters" tab, then click on the plus icon and select "Input Parameter"
    ![New Input Parameter Step 1](./img/Ex3_Add_Input_Param1.png)

    1. The Input Parameter currently has the generated name of "IP_1" which we can change by clicking on the `>` icon to the right of the screen
    ![New Input Parameter Step 2](./img/Ex3_Add_Input_Param2.png)

    1. Enter the name, data type and length properties from the table above, and select the "Is Mandatory" flag
    ![New Input Parameter Step 3](./img/Ex3_Add_Input_Param3.png)

    1. Click on "Back" and add the second input parameter
    ![New Input Parameter Step 4](./img/Ex3_Add_Input_Param4.png)


1. Now we need to take the values received as input parameters and use them to filter the table data.  This is where a filter expression is used.

    Select the "Filter Expression" tab from the Projection editor

   ![Add Filter Step 1](./img/Ex3_Add_Filter1.png)

1. Paste the following condition into the Expression text area

    ```
    "STARTINGAIRPORT_IATA3" = '$$airportFrom$$' 
    AND 
    "DESTINATIONAIRPORT_IATA3" = '$$airportTo$$'
    ```

    Here we have created a filter condition that will be applied to every row of the table.  Notice the "double dollar" delimiters used to distinguish an input parameter name
    
    Then press Validate Syntax
    
    ![Validate Syntax](./img/Ex3_Add_Filter2.png)
    
    You should see a pop-up informing you that the expression is valid
    
    ![Validate Syntax](./img/Ex3_Valid_Syntax.png)

1.  Connect our new `Projection_1` to the Calculation View's projection.  This is done by dragging the "Connect" icon of `Projection_1` up to the main projection

    ![Add Projection 1](./img/Ex3_Add_Projection1.png)

    Our new projection is now connected

    ![Add Projection 2](./img/Ex3_Add_Projection2.png)

1.  Click on the main Projection and select the "Mapping" tab at the top of the detail pane.
    
    Notice that `Projection_1` is now the data source rather than the underlying database table. This means that the main projection can only see those fields that have been "projected" through `Projection_1` from the underlying database table.
    
    So the purpose of `Projection_1` is to apply various filters and conditions that restrict the rows and columns supplied to the main Projection.  In this case, we are restricting only the rows to those that match the input parameter values: we not restricting the columns.
    
    Drag all of `Projection_1` onto the Output Columns area.
    
    ![Add Projection Mapping - Before](./img/Ex3_Add_Projection_Mapping_Before.png)

    The screen will now look like this:
    
    ![Add Projection Mapping - After](./img/Ex3_Add_Projection_Mapping_After.png)

1. The last step in defining the calculation view is to create a "Calculated Column".

    Here, we will create a column to appear in the calculation view whose content has been constructed dynamically.  In this case, we will create a column that contains a text string constructed from the IATA location codes of the starting and destination airports.

    Select the "Calculated Columns" tab from the detail pane and then press the `+` sign to add a new calculated column.
    
    ![Add Calculated Column 1](./img/Ex3_Calc_Col1.png)

    ![Add Calculated Column 2](./img/Ex3_Calc_Col2.png)

1. Call the calculated column `routeText` and change it to be of type `NVARCHAR` and have a length of `50`

    ![Add Calculated Column 3](./img/Ex3_Calc_Col3.png)

1. Expand the "Expression" section and select the Expression Editor

    ![Add Calculated Column 4](./img/Ex3_Calc_Col4.png)

1.  Paste the following text into the expression editor:
   
    ```
    CONCAT(
      CONCAT(
        "STARTINGAIRPORT_IATA3"
      , '->'
      )
    , "DESTINATIONAIRPORT_IATA3"
    )
    ```

    Due to the fact that the SQL function `CONCAT` can only take two parameters, we must use nested `CONCAT` functions. The value of the Starting Airport is concatenated to "`->`", which in turn is concatenated to the value of the Destination Airport.
    
    Now press "Validate Syntax" and you should see that the syntax checker is happy.

    ![Validate Syntax](./img/Ex3_Valid_Syntax.png)

1. Click on "Semantics" at the top of the graphical display of the calculation view.

    ![Semantics](./img/Ex3_Semantics.png)
    
    In the detail pane, you can now see all the columns that will make up the final table visible in the calculation view.  This includes the new calculated column called `routeText`

1. Build the calculation view

    ![Build the calculation view](./img/Ex3_Build_Calc_View.png)

1. In order to display information through our calculation view, right-click on the calculation view and select Data Preview

    ![Data Preview](./img/Ex3_Data_Preview1.png)

1. In the `airportFrom` and `airportTo` fields, enter the 3-character location codes of your desired route.  For instance you can search for direct flights from John F. Kennedy Airport in New York (`JFK`) to Las Vegas (`LAS`)

    ***Important***  
    These input parameter values are case-sensitive!

    ![Data Preview](./img/Ex3_Data_Preview2.png)

    Once you have entered both input parameter values (and checked that the second value has not overwritten the first value), press the "Open Content" button circled at the top of the screen shot.

1. You will now see a display of all the carriers that operate direct flights between John F. Kennedy airport in New York and Las Vegas (5 carriers in this case)

    ![Calculation View Results](./img/Ex3_Calc_View_Results.png)

    If for some reason you see no results, then one of the following situations could have occurred:
    
    1. The `airportFrom` and `airportTo` fields contain the same value; or
    1. You have entered one or both of the 3-character IATA location codes in lowercase instead of uppercase; or
    1. You have entered two location codes between which there are genuinely no direct flights.  For instance, no carrier flies directly from Madrid in Spain (`MAD`) to Las Vegas (`LAS`)



[Previous Exercise](./ex2_shortest_path.md)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[Next Exercise](./ex4_one_stop_calc_view.md)


# \</exercise>
