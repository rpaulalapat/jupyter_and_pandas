
# Schools report

# Conclusions

1. There is significant variance in pass rates between schools
2. Pass rates do not vary significantly between grades within the same school
3. Spending per student does not seem to have the effect expected (more spending does not result in higher passing rates). Infact, it looks like higher funding lowers passing rate, hence funding should be reduced. However, this assesment needs further analysis by considering other factors that possibly have a higher impact. For example, does funding have an effect within a particular the school type. On doing this, we can make two conslusions. First, within a certain school type, where funding is similar, there is not much difference. Second, that Charter schools, which have comparitively lower funding per student, do much better in terms of passing rate. Hence it is concluding that the type of school has a higher impact, but funding as such does not (lowering funding for any given school is not going to improve pass rate).
4. School size has a negative effect on pass rates above a threshhold (~ 2000)
5. School type has a significant effect on pass rates, with charter schools performing much better than district schoolsd 
6. Recommended actions to improve passing rate based on above analysis is add more charter schools, and reduce school size to less than 2000 students. Budget for this could be partially realized by reducing the funding for highly funded schools and ensuring all schools have a similar funding.


```python
# import modules
import pandas as pd
```


```python
# read data
file_schools = 'raw_data/schools_complete.csv'
file_students = 'raw_data/students_complete.csv'

schools_df = pd.read_csv(file_schools)
students_df = pd.read_csv(file_students)
```


```python
# analyze data check for issues or if cleanup is needed
#schools_df.head()

#students_df.head()

#check for nulls
#schools_df.count()
#schools_df.columns
#students_df.columns

#students_df.count()
# no nulls
```

# District summary


```python
#Total Schools
total_schools_count = schools_df.count()['School ID']
#total_schools_count

#Total Students 
total_students_count = students_df.count()["Student ID"]

#Total Budget
total_budget = schools_df['budget'].sum()

#Average Math Score
avg_math_score = students_df['math_score'].mean()
#Average Reading Score
avg_reading_score = students_df['reading_score'].mean()
# % Passing Math
pass_math_df = students_df.loc[(students_df["math_score"] >= 70),:]
pass_math_count = pass_math_df.count()['math_score']
pass_math_pc = round(pass_math_count/total_students_count*100,2)

# % Passing Reading
pass_reading_df = students_df.loc[(students_df["reading_score"] >= 70),:]
pass_reading_count = pass_math_df.count()['reading_score']
pass_reading_pc = round(pass_reading_count/total_students_count*100,2)

# Overall Passing Rate (Average of the above two)
overall_pass_rate = round((pass_reading_pc + pass_math_pc)/2,2)

results_df = pd.DataFrame([{
                            'Total Schools':total_schools_count,
                            'Total Students':total_students_count,
                            'Total Budget':total_budget,
                            'Average Math Score':avg_math_score,
                            'Average Reading Score':avg_reading_score,
                            '% Passing Math':pass_math_pc,
                            '% Passing Reading':pass_reading_pc,
                            'Overall Passing Rate':overall_pass_rate,
                        }])
results_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>Overall Passing Rate</th>
      <th>Total Budget</th>
      <th>Total Schools</th>
      <th>Total Students</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>74.98</td>
      <td>74.98</td>
      <td>78.985371</td>
      <td>81.87784</td>
      <td>74.98</td>
      <td>24649428</td>
      <td>15</td>
      <td>39170</td>
    </tr>
  </tbody>
</table>
</div>



# School summary


```python
#School summary
#School Name
#School Type
#Total Students
#Total School Budget
#Per Student Budget
#Average Math Score
#Average Reading Score
#% Passing Math
#% Passing Reading
#Overall Passing Rate 


schools_summary_df = schools_df[['name','type','size','budget']]
schools_summary_df.rename(columns = {'name':'School Name',
                                    'type':'School Type',
                                    'size':'Total Students',
                                    'budget':'Total School Budget'}, inplace = True)
schools_summary_df["Per Student Budget"] = (schools_summary_df['Total School Budget'] / schools_summary_df['Total Students'])

#set index so that other series with same index can be added
schools_summary_df.set_index('School Name', inplace=True)

# group df by school and calculate values
grouped_students_by_school = students_df.groupby(['school'])

school_math_avg = grouped_students_by_school['math_score'].mean()
school_reading_avg = grouped_students_by_school['reading_score'].mean()
schools_summary_df['Average Math Score'] = school_math_avg
schools_summary_df['Average Reading Score'] = school_reading_avg


pass_math_df = students_df[students_df["math_score"] >= 70]
pass_math_by_school_df = pass_math_df.groupby('school')['math_score'].size()
pass_reading_df = students_df[students_df["reading_score"] >= 70]
pass_reading_by_school_df = pass_reading_df.groupby('school')['reading_score'].size()

schools_summary_df['Math_pass_count'] = pass_math_by_school_df
schools_summary_df['Reading_pass_count'] = pass_reading_by_school_df

schools_summary_df['% Passing Math'] = (schools_summary_df['Math_pass_count']/schools_summary_df['Total Students'])*100
schools_summary_df['% Passing Reading'] = (schools_summary_df['Reading_pass_count']/schools_summary_df['Total Students'])*100
schools_summary_df['Overall Passing Rate'] = (schools_summary_df['% Passing Math'] + schools_summary_df['% Passing Reading']) / 2

#create final display df with needed columns and formats

schools_summary_display_df = schools_summary_df[['School Type','Total Students','Total School Budget','Per Student Budget','Average Math Score',
                               'Average Reading Score','% Passing Math','% Passing Reading','Overall Passing Rate']]

#format columns appropriately
schools_summary_display_df["Per Student Budget"] = schools_summary_display_df["Per Student Budget"].map("${:.2f}".format)
schools_summary_display_df['Average Math Score'] = schools_summary_display_df['Average Math Score'].map("{:.2f}".format)
schools_summary_display_df['Average Reading Score'] = schools_summary_display_df['Average Reading Score'].map("{:.2f}".format)
schools_summary_display_df['% Passing Math'] = schools_summary_display_df['% Passing Math'].map("{:.2f}".format)
schools_summary_display_df['% Passing Reading'] = schools_summary_display_df['% Passing Reading'].map("{:.2f}".format)
schools_summary_display_df['Overall Passing Rate'] = schools_summary_display_df['Overall Passing Rate'].map("{:.2f}".format)


schools_summary_display_df

```

    C:\Users\rpaul\Anaconda3\lib\site-packages\pandas\core\frame.py:3027: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame
    
    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/indexing.html#indexing-view-versus-copy
      return super(DataFrame, self).rename(**kwargs)
    C:\Users\rpaul\Anaconda3\lib\site-packages\ipykernel\__main__.py:51: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead
    
    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/indexing.html#indexing-view-versus-copy
    C:\Users\rpaul\Anaconda3\lib\site-packages\ipykernel\__main__.py:52: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead
    
    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/indexing.html#indexing-view-versus-copy
    C:\Users\rpaul\Anaconda3\lib\site-packages\ipykernel\__main__.py:53: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead
    
    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/indexing.html#indexing-view-versus-copy
    C:\Users\rpaul\Anaconda3\lib\site-packages\ipykernel\__main__.py:54: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead
    
    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/indexing.html#indexing-view-versus-copy
    C:\Users\rpaul\Anaconda3\lib\site-packages\ipykernel\__main__.py:55: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead
    
    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/indexing.html#indexing-view-versus-copy
    C:\Users\rpaul\Anaconda3\lib\site-packages\ipykernel\__main__.py:56: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead
    
    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/indexing.html#indexing-view-versus-copy
    




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total School Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
    </tr>
    <tr>
      <th>School Name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Huang High School</th>
      <td>District</td>
      <td>2917</td>
      <td>1910635</td>
      <td>$655.00</td>
      <td>76.63</td>
      <td>81.18</td>
      <td>65.68</td>
      <td>81.32</td>
      <td>73.50</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>District</td>
      <td>2949</td>
      <td>1884411</td>
      <td>$639.00</td>
      <td>76.71</td>
      <td>81.16</td>
      <td>65.99</td>
      <td>80.74</td>
      <td>73.36</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>Charter</td>
      <td>1761</td>
      <td>1056600</td>
      <td>$600.00</td>
      <td>83.36</td>
      <td>83.73</td>
      <td>93.87</td>
      <td>95.85</td>
      <td>94.86</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>District</td>
      <td>4635</td>
      <td>3022020</td>
      <td>$652.00</td>
      <td>77.29</td>
      <td>80.93</td>
      <td>66.75</td>
      <td>80.86</td>
      <td>73.81</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>Charter</td>
      <td>1468</td>
      <td>917500</td>
      <td>$625.00</td>
      <td>83.35</td>
      <td>83.82</td>
      <td>93.39</td>
      <td>97.14</td>
      <td>95.27</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>Charter</td>
      <td>2283</td>
      <td>1319574</td>
      <td>$578.00</td>
      <td>83.27</td>
      <td>83.99</td>
      <td>93.87</td>
      <td>96.54</td>
      <td>95.20</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>Charter</td>
      <td>1858</td>
      <td>1081356</td>
      <td>$582.00</td>
      <td>83.06</td>
      <td>83.98</td>
      <td>94.13</td>
      <td>97.04</td>
      <td>95.59</td>
    </tr>
    <tr>
      <th>Bailey High School</th>
      <td>District</td>
      <td>4976</td>
      <td>3124928</td>
      <td>$628.00</td>
      <td>77.05</td>
      <td>81.03</td>
      <td>66.68</td>
      <td>81.93</td>
      <td>74.31</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>Charter</td>
      <td>427</td>
      <td>248087</td>
      <td>$581.00</td>
      <td>83.80</td>
      <td>83.81</td>
      <td>92.51</td>
      <td>96.25</td>
      <td>94.38</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>Charter</td>
      <td>962</td>
      <td>585858</td>
      <td>$609.00</td>
      <td>83.84</td>
      <td>84.04</td>
      <td>94.59</td>
      <td>95.95</td>
      <td>95.27</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>Charter</td>
      <td>1800</td>
      <td>1049400</td>
      <td>$583.00</td>
      <td>83.68</td>
      <td>83.95</td>
      <td>93.33</td>
      <td>96.61</td>
      <td>94.97</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>District</td>
      <td>3999</td>
      <td>2547363</td>
      <td>$637.00</td>
      <td>76.84</td>
      <td>80.74</td>
      <td>66.37</td>
      <td>80.22</td>
      <td>73.29</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>District</td>
      <td>4761</td>
      <td>3094650</td>
      <td>$650.00</td>
      <td>77.07</td>
      <td>80.97</td>
      <td>66.06</td>
      <td>81.22</td>
      <td>73.64</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>District</td>
      <td>2739</td>
      <td>1763916</td>
      <td>$644.00</td>
      <td>77.10</td>
      <td>80.75</td>
      <td>68.31</td>
      <td>79.30</td>
      <td>73.80</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>Charter</td>
      <td>1635</td>
      <td>1043130</td>
      <td>$638.00</td>
      <td>83.42</td>
      <td>83.85</td>
      <td>93.27</td>
      <td>97.31</td>
      <td>95.29</td>
    </tr>
  </tbody>
</table>
</div>



# Top Performing Schools (By Passing Rate)


```python
#display top 5 performing schools (by passing rate)

# sort the schools summary df by passing rate (Overall Passing Rate) in descending order

schools_summary_sorted_df = schools_summary_display_df.sort_values(by='Overall Passing Rate', ascending = False)

#display first 5
schools_summary_sorted_df.head(5)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total School Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
    </tr>
    <tr>
      <th>School Name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Cabrera High School</th>
      <td>Charter</td>
      <td>1858</td>
      <td>1081356</td>
      <td>$582.00</td>
      <td>83.06</td>
      <td>83.98</td>
      <td>94.13</td>
      <td>97.04</td>
      <td>95.59</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>Charter</td>
      <td>1635</td>
      <td>1043130</td>
      <td>$638.00</td>
      <td>83.42</td>
      <td>83.85</td>
      <td>93.27</td>
      <td>97.31</td>
      <td>95.29</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>Charter</td>
      <td>1468</td>
      <td>917500</td>
      <td>$625.00</td>
      <td>83.35</td>
      <td>83.82</td>
      <td>93.39</td>
      <td>97.14</td>
      <td>95.27</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>Charter</td>
      <td>962</td>
      <td>585858</td>
      <td>$609.00</td>
      <td>83.84</td>
      <td>84.04</td>
      <td>94.59</td>
      <td>95.95</td>
      <td>95.27</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>Charter</td>
      <td>2283</td>
      <td>1319574</td>
      <td>$578.00</td>
      <td>83.27</td>
      <td>83.99</td>
      <td>93.87</td>
      <td>96.54</td>
      <td>95.20</td>
    </tr>
  </tbody>
</table>
</div>



# Lowest Performing Schools (by Pass rate)


```python
# display the last 5 schools from the sorted df
schools_summary_sorted_df.tail(5)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total School Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
    </tr>
    <tr>
      <th>School Name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Ford High School</th>
      <td>District</td>
      <td>2739</td>
      <td>1763916</td>
      <td>$644.00</td>
      <td>77.10</td>
      <td>80.75</td>
      <td>68.31</td>
      <td>79.30</td>
      <td>73.80</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>District</td>
      <td>4761</td>
      <td>3094650</td>
      <td>$650.00</td>
      <td>77.07</td>
      <td>80.97</td>
      <td>66.06</td>
      <td>81.22</td>
      <td>73.64</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>District</td>
      <td>2917</td>
      <td>1910635</td>
      <td>$655.00</td>
      <td>76.63</td>
      <td>81.18</td>
      <td>65.68</td>
      <td>81.32</td>
      <td>73.50</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>District</td>
      <td>2949</td>
      <td>1884411</td>
      <td>$639.00</td>
      <td>76.71</td>
      <td>81.16</td>
      <td>65.99</td>
      <td>80.74</td>
      <td>73.36</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>District</td>
      <td>3999</td>
      <td>2547363</td>
      <td>$637.00</td>
      <td>76.84</td>
      <td>80.74</td>
      <td>66.37</td>
      <td>80.22</td>
      <td>73.29</td>
    </tr>
  </tbody>
</table>
</div>



# Math Scores by Grade


```python
# Create a table that lists the average Math Score for students of each grade level (9th, 10th, 11th, 12th) at each school.

# get series for each grade, then group by school and calculate mean (to have grades as columns)

grade9_m_score = students_df[students_df['grade'] == '9th']
avg_grade9_m_score_by_school = grade9_m_score.groupby('school')['math_score'].mean()
grade10_m_score = students_df[students_df['grade'] == '10th']
avg_grade10_m_score_by_school = grade10_m_score.groupby('school')['math_score'].mean()
grade11_m_score = students_df[students_df['grade'] == '11th']
avg_grade11_m_score_by_school = grade11_m_score.groupby('school')['math_score'].mean()
grade12_m_score = students_df[students_df['grade'] == '12th']
avg_grade12_m_score_by_school = grade12_m_score.groupby('school')['math_score'].mean()

#create new DF for display
avg_mscore_by_grade_table = pd.DataFrame({'09th': avg_grade9_m_score_by_school,
                                    '10th': avg_grade10_m_score_by_school,
                                   '11th': avg_grade11_m_score_by_school,
                                   '12th': avg_grade12_m_score_by_school})
# Display
avg_mscore_by_grade_table
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>09th</th>
      <th>10th</th>
      <th>11th</th>
      <th>12th</th>
    </tr>
    <tr>
      <th>school</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Bailey High School</th>
      <td>77.083676</td>
      <td>76.996772</td>
      <td>77.515588</td>
      <td>76.492218</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>83.094697</td>
      <td>83.154506</td>
      <td>82.765560</td>
      <td>83.277487</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>76.403037</td>
      <td>76.539974</td>
      <td>76.884344</td>
      <td>77.151369</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>77.361345</td>
      <td>77.672316</td>
      <td>76.918058</td>
      <td>76.179963</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>82.044010</td>
      <td>84.229064</td>
      <td>83.842105</td>
      <td>83.356164</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>77.438495</td>
      <td>77.337408</td>
      <td>77.136029</td>
      <td>77.186567</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>83.787402</td>
      <td>83.429825</td>
      <td>85.000000</td>
      <td>82.855422</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>77.027251</td>
      <td>75.908735</td>
      <td>76.446602</td>
      <td>77.225641</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>77.187857</td>
      <td>76.691117</td>
      <td>77.491653</td>
      <td>76.863248</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>83.625455</td>
      <td>83.372000</td>
      <td>84.328125</td>
      <td>84.121547</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>76.859966</td>
      <td>76.612500</td>
      <td>76.395626</td>
      <td>77.690748</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>83.420755</td>
      <td>82.917411</td>
      <td>83.383495</td>
      <td>83.778976</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>83.590022</td>
      <td>83.087886</td>
      <td>83.498795</td>
      <td>83.497041</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>83.085578</td>
      <td>83.724422</td>
      <td>83.195326</td>
      <td>83.035794</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>83.264706</td>
      <td>84.010288</td>
      <td>83.836782</td>
      <td>83.644986</td>
    </tr>
  </tbody>
</table>
</div>



# Reading Score by Grade


```python
# Create a table that lists the average Math Score for students of each grade level (9th, 10th, 11th, 12th) at each school.

# get series for each grade, then group by school and calculate mean (to have grades as columns)

grade9_r_score = students_df[students_df['grade'] == '9th']
avg_grade9_r_score_by_school = grade9_r_score.groupby('school')['reading_score'].mean()
grade10_r_score = students_df[students_df['grade'] == '10th']
avg_grade10_r_score_by_school = grade10_r_score.groupby('school')['reading_score'].mean()
grade11_r_score = students_df[students_df['grade'] == '11th']
avg_grade11_r_score_by_school = grade11_r_score.groupby('school')['reading_score'].mean()
grade12_r_score = students_df[students_df['grade'] == '12th']
avg_grade12_r_score_by_school = grade12_r_score.groupby('school')['reading_score'].mean()

#create new DF for display
avg_rscore_by_grade_table = pd.DataFrame({'09th': avg_grade9_r_score_by_school,
                                    '10th': avg_grade10_r_score_by_school,
                                   '11th': avg_grade11_r_score_by_school,
                                   '12th': avg_grade12_r_score_by_school})
# Display
avg_rscore_by_grade_table

```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>09th</th>
      <th>10th</th>
      <th>11th</th>
      <th>12th</th>
    </tr>
    <tr>
      <th>school</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Bailey High School</th>
      <td>81.303155</td>
      <td>80.907183</td>
      <td>80.945643</td>
      <td>80.912451</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>83.676136</td>
      <td>84.253219</td>
      <td>83.788382</td>
      <td>84.287958</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>81.198598</td>
      <td>81.408912</td>
      <td>80.640339</td>
      <td>81.384863</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>80.632653</td>
      <td>81.262712</td>
      <td>80.403642</td>
      <td>80.662338</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>83.369193</td>
      <td>83.706897</td>
      <td>84.288089</td>
      <td>84.013699</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>80.866860</td>
      <td>80.660147</td>
      <td>81.396140</td>
      <td>80.857143</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>83.677165</td>
      <td>83.324561</td>
      <td>83.815534</td>
      <td>84.698795</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>81.290284</td>
      <td>81.512386</td>
      <td>81.417476</td>
      <td>80.305983</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>81.260714</td>
      <td>80.773431</td>
      <td>80.616027</td>
      <td>81.227564</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>83.807273</td>
      <td>83.612000</td>
      <td>84.335938</td>
      <td>84.591160</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>80.993127</td>
      <td>80.629808</td>
      <td>80.864811</td>
      <td>80.376426</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>84.122642</td>
      <td>83.441964</td>
      <td>84.373786</td>
      <td>82.781671</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>83.728850</td>
      <td>84.254157</td>
      <td>83.585542</td>
      <td>83.831361</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>83.939778</td>
      <td>84.021452</td>
      <td>83.764608</td>
      <td>84.317673</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>83.833333</td>
      <td>83.812757</td>
      <td>84.156322</td>
      <td>84.073171</td>
    </tr>
  </tbody>
</table>
</div>



# Scores by School Spending


```python
# Create a table that breaks down school performances based on average Spending Ranges (Per Student). Use 4 reasonable bins to group school spending. Include in the table each of the following:
  # Average Math Score
  # Average Reading Score
  # % Passing Math
  # % Passing Reading
  # Overall Passing Rate (Average of the above two)

# Create the bins in which Data will be held
# Bins are 575 to 600, 600 to 625, 625 to 650, 650 to 675
bins = [0, 585, 615, 645, 675]

# Create the names for the four bins
group_names = ['<585', '$585-615', '$615-645', '$645-675']

#create a merged df of school and students
score_school_spend_df = schools_df[['name','size','budget']]
score_school_spend_df.rename(columns = {'name':'School Name',
                                    'size':'Total Students',
                                    'budget':'Total School Budget'}, inplace = True)
score_school_spend_df["Per Student Budget"] = (score_school_spend_df['Total School Budget'] / score_school_spend_df['Total Students'])
score_students_df = students_df[['school','Student ID','math_score','reading_score']]
score_students_df.rename(columns={'school':'School Name'}, inplace=True)
merged_score_school_spend_df = pd.merge(score_school_spend_df,score_students_df, on = 'School Name')
summary_score_school_spend_df = merged_score_school_spend_df[['School Name','Per Student Budget','Student ID','math_score','reading_score']]

# Cut postTestScore and place the scores into bins
summary_score_school_spend_df['Spending Ranges(Per Student)']=pd.cut(summary_score_school_spend_df['Per Student Budget'], bins, labels=group_names)

grouped_students_by_score = summary_score_school_spend_df.groupby('Spending Ranges(Per Student)')

score_math_avg = grouped_students_by_score['math_score'].mean()
score_reading_avg = grouped_students_by_score['reading_score'].mean()
score_total_students = grouped_students_by_score['Student ID'].size()

pass_math_df = summary_score_school_spend_df[summary_score_school_spend_df["math_score"] >= 70]
pass_math_by_score_df = pass_math_df.groupby('Spending Ranges(Per Student)')['math_score'].size()
pass_reading_df =  summary_score_school_spend_df[summary_score_school_spend_df["reading_score"] >= 70]
pass_reading_by_score_df =  pass_reading_df.groupby('Spending Ranges(Per Student)')['reading_score'].size()


#create new dataframe with calculated values
score_summary_table = pd.DataFrame({'Average Math Score': score_math_avg,
                                    'Average Reading Score': score_reading_avg,
                                   'Total Students': score_total_students,
                                   'Math_pass_count': pass_math_by_score_df,
                                   'Reading_pass_count': pass_reading_by_score_df})
score_summary_table['% Passing Math'] = (score_summary_table['Math_pass_count']/score_summary_table['Total Students'])*100
score_summary_table['% Passing Reading'] = (score_summary_table['Reading_pass_count']/score_summary_table['Total Students'])*100
score_summary_table['Overall Passing Rate'] = (score_summary_table['% Passing Math'] + score_summary_table['% Passing Reading']) / 2

#drop columns not needed
score_summary_table = score_summary_table.drop(score_summary_table.columns[[2,3,4]], axis=1)

#Display
score_summary_table
```

    C:\Users\rpaul\Anaconda3\lib\site-packages\pandas\core\frame.py:3027: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame
    
    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/indexing.html#indexing-view-versus-copy
      return super(DataFrame, self).rename(**kwargs)
    C:\Users\rpaul\Anaconda3\lib\site-packages\ipykernel\__main__.py:20: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead
    
    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/indexing.html#indexing-view-versus-copy
    C:\Users\rpaul\Anaconda3\lib\site-packages\ipykernel\__main__.py:27: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead
    
    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/indexing.html#indexing-view-versus-copy
    




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
    </tr>
    <tr>
      <th>Spending Ranges(Per Student)</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>&lt;585</th>
      <td>83.363065</td>
      <td>83.964039</td>
      <td>93.702889</td>
      <td>96.686558</td>
      <td>95.194724</td>
    </tr>
    <tr>
      <th>$585-615</th>
      <td>83.529196</td>
      <td>83.838414</td>
      <td>94.124128</td>
      <td>95.886889</td>
      <td>95.005509</td>
    </tr>
    <tr>
      <th>$615-645</th>
      <td>78.061635</td>
      <td>81.434088</td>
      <td>71.400428</td>
      <td>83.614770</td>
      <td>77.507599</td>
    </tr>
    <tr>
      <th>$645-675</th>
      <td>77.049297</td>
      <td>81.005604</td>
      <td>66.230813</td>
      <td>81.109397</td>
      <td>73.670105</td>
    </tr>
  </tbody>
</table>
</div>



# Scores by School Size


```python
# group schools based on a reasonable approximation of school size (Small < 1000, Medium 1000-2000, Large 2000-5000)

# Create the bins in which Data will be held
# Bins are Small < 1000, Medium 1000-2000, Large 2000-5000
bins = [0, 1000, 2000, 5000]

# Create the names for the four bins
group_names = ['Small (< 1000)', 'Medium (1000-2000)', 'Large (>2000)']

#create a merged df of school and students
score_school_size_df = schools_df[['name','size','budget']]
score_school_size_df.rename(columns = {'name':'School Name',
                                    'size':'Total Students',
                                    'budget':'Total School Budget'}, inplace = True)

score_students_df = students_df[['school','Student ID','math_score','reading_score']]
score_students_df.rename(columns={'school':'School Name'}, inplace=True)
merged_score_school_size_df = pd.merge(score_school_size_df,score_students_df, on = 'School Name')
summary_score_school_size_df = merged_score_school_size_df[['School Name','Total Students','Student ID','math_score','reading_score']]

# Cut postTestScore and place the scores into bins
summary_score_school_size_df['School size']=pd.cut(summary_score_school_size_df['Total Students'], bins, labels=group_names)

# group by school size and calculate values
grouped_students_by_score = summary_score_school_size_df.groupby('School size')

score_math_avg = grouped_students_by_score['math_score'].mean()
score_reading_avg = grouped_students_by_score['reading_score'].mean()
score_total_students = grouped_students_by_score['Student ID'].size()

pass_math_df = summary_score_school_size_df[summary_score_school_size_df["math_score"] >= 70]
pass_math_by_score_df = pass_math_df.groupby('School size')['math_score'].size()
pass_reading_df =  summary_score_school_size_df[summary_score_school_size_df["reading_score"] >= 70]
pass_reading_by_score_df =  pass_reading_df.groupby('School size')['reading_score'].size()


#create new dataframe with calculated values
score_summary_table = pd.DataFrame({'Average Math Score': score_math_avg,
                                    'Average Reading Score': score_reading_avg,
                                   'Total Students': score_total_students,
                                   'Math_pass_count': pass_math_by_score_df,
                                   'Reading_pass_count': pass_reading_by_score_df})
score_summary_table['% Passing Math'] = (score_summary_table['Math_pass_count']/score_summary_table['Total Students'])*100
score_summary_table['% Passing Reading'] = (score_summary_table['Reading_pass_count']/score_summary_table['Total Students'])*100
score_summary_table['Overall Passing Rate'] = (score_summary_table['% Passing Math'] + score_summary_table['% Passing Reading']) / 2

#drop columns not needed
score_summary_table = score_summary_table.drop(score_summary_table.columns[[2,3,4]], axis=1)

#Display
score_summary_table
```

    C:\Users\rpaul\Anaconda3\lib\site-packages\pandas\core\frame.py:3027: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame
    
    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/indexing.html#indexing-view-versus-copy
      return super(DataFrame, self).rename(**kwargs)
    




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
    </tr>
    <tr>
      <th>School size</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Small (&lt; 1000)</th>
      <td>83.828654</td>
      <td>83.974082</td>
      <td>93.952484</td>
      <td>96.040317</td>
      <td>94.996400</td>
    </tr>
    <tr>
      <th>Medium (1000-2000)</th>
      <td>83.372682</td>
      <td>83.867989</td>
      <td>93.616522</td>
      <td>96.773058</td>
      <td>95.194790</td>
    </tr>
    <tr>
      <th>Large (&gt;2000)</th>
      <td>77.477597</td>
      <td>81.198674</td>
      <td>68.652380</td>
      <td>82.125158</td>
      <td>75.388769</td>
    </tr>
  </tbody>
</table>
</div>



# Scores by School Type


```python
# Repeat the above breakdown, but this time group schools based on school type (Charter vs. District).

#create a merged df of school and students
score_school_type_df = schools_df[['name','type']]
score_school_type_df.rename(columns = {'name':'School Name',
                                        'type':'School type'
                                        }, inplace = True)

score_students_df = students_df[['school','Student ID','math_score','reading_score']]
score_students_df.rename(columns={'school':'School Name'}, inplace=True)
merged_score_school_type_df = pd.merge(score_school_type_df,score_students_df, on = 'School Name')
summary_score_school_type_df = merged_score_school_type_df[['School Name','School type','Student ID','math_score','reading_score']]

# group by School Type and calculate values
grouped_students_by_score = summary_score_school_type_df.groupby('School type')

score_math_avg = grouped_students_by_score['math_score'].mean()
score_reading_avg = grouped_students_by_score['reading_score'].mean()
score_total_students = grouped_students_by_score['Student ID'].size()

pass_math_df = summary_score_school_type_df[summary_score_school_type_df["math_score"] >= 70]
pass_math_by_score_df = pass_math_df.groupby('School type')['math_score'].size()
pass_reading_df =  summary_score_school_type_df[summary_score_school_type_df["reading_score"] >= 70]
pass_reading_by_score_df =  pass_reading_df.groupby('School type')['reading_score'].size()


#create new dataframe with calculated values
score_summary_table = pd.DataFrame({'Average Math Score': score_math_avg,
                                    'Average Reading Score': score_reading_avg,
                                   'Total Students': score_total_students,
                                   'Math_pass_count': pass_math_by_score_df,
                                   'Reading_pass_count': pass_reading_by_score_df})
score_summary_table['% Passing Math'] = (score_summary_table['Math_pass_count']/score_summary_table['Total Students'])*100
score_summary_table['% Passing Reading'] = (score_summary_table['Reading_pass_count']/score_summary_table['Total Students'])*100
score_summary_table['Overall Passing Rate'] = (score_summary_table['% Passing Math'] + score_summary_table['% Passing Reading']) / 2

#drop columns not needed
score_summary_table = score_summary_table.drop(score_summary_table.columns[[2,3,4]], axis=1)

#Display
score_summary_table
```

    C:\Users\rpaul\Anaconda3\lib\site-packages\pandas\core\frame.py:3027: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame
    
    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/indexing.html#indexing-view-versus-copy
      return super(DataFrame, self).rename(**kwargs)
    




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
    </tr>
    <tr>
      <th>School type</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Charter</th>
      <td>83.406183</td>
      <td>83.902821</td>
      <td>93.701821</td>
      <td>96.645891</td>
      <td>95.173856</td>
    </tr>
    <tr>
      <th>District</th>
      <td>76.987026</td>
      <td>80.962485</td>
      <td>66.518387</td>
      <td>80.905249</td>
      <td>73.711818</td>
    </tr>
  </tbody>
</table>
</div>



# Scores by School Spending for Charter schools


```python
# Create a table that breaks down school performances based on average Spending Ranges (Per Student), only for Charter schools

# Create the bins in which Data will be held
# Bins are 575 to 600, 600 to 625, 625 to 650, 650 to 675
bins = [0, 585, 615, 645, 675]

# Create the names for the four bins
group_names = ['<585', '$585-615', '$615-645', '$645-675']

#create a merged df of school and students
score_school_spend_df = schools_df[schools_df['type'] == "Charter"]
score_school_spend_df.rename(columns = {'name':'School Name',
                                    'size':'Total Students',
                                    'budget':'Total School Budget'}, inplace = True)
score_school_spend_df["Per Student Budget"] = (score_school_spend_df['Total School Budget'] / score_school_spend_df['Total Students'])
score_students_df = students_df[['school','Student ID','math_score','reading_score']]
score_students_df.rename(columns={'school':'School Name'}, inplace=True)
merged_score_school_spend_df = pd.merge(score_school_spend_df,score_students_df, on = 'School Name')
summary_score_school_spend_df = merged_score_school_spend_df[['School Name','Per Student Budget','Student ID','math_score','reading_score']]

# Cut postTestScore and place the scores into bins
summary_score_school_spend_df['Spending Ranges(Per Student)']=pd.cut(summary_score_school_spend_df['Per Student Budget'], bins, labels=group_names)

grouped_students_by_score = summary_score_school_spend_df.groupby('Spending Ranges(Per Student)')

score_math_avg = grouped_students_by_score['math_score'].mean()
score_reading_avg = grouped_students_by_score['reading_score'].mean()
score_total_students = grouped_students_by_score['Student ID'].size()

pass_math_df = summary_score_school_spend_df[summary_score_school_spend_df["math_score"] >= 70]
pass_math_by_score_df = pass_math_df.groupby('Spending Ranges(Per Student)')['math_score'].size()
pass_reading_df =  summary_score_school_spend_df[summary_score_school_spend_df["reading_score"] >= 70]
pass_reading_by_score_df =  pass_reading_df.groupby('Spending Ranges(Per Student)')['reading_score'].size()


#create new dataframe with calculated values
score_summary_table = pd.DataFrame({'Average Math Score': score_math_avg,
                                    'Average Reading Score': score_reading_avg,
                                   'Total Students': score_total_students,
                                   'Math_pass_count': pass_math_by_score_df,
                                   'Reading_pass_count': pass_reading_by_score_df})
score_summary_table['% Passing Math'] = (score_summary_table['Math_pass_count']/score_summary_table['Total Students'])*100
score_summary_table['% Passing Reading'] = (score_summary_table['Reading_pass_count']/score_summary_table['Total Students'])*100
score_summary_table['Overall Passing Rate'] = (score_summary_table['% Passing Math'] + score_summary_table['% Passing Reading']) / 2

#drop columns not needed
score_summary_table = score_summary_table.drop(score_summary_table.columns[[2,3,4]], axis=1)

#Display
score_summary_table
```

    C:\Users\rpaul\Anaconda3\lib\site-packages\pandas\core\frame.py:3027: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame
    
    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/indexing.html#indexing-view-versus-copy
      return super(DataFrame, self).rename(**kwargs)
    C:\Users\rpaul\Anaconda3\lib\site-packages\ipykernel\__main__.py:15: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead
    
    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/indexing.html#indexing-view-versus-copy
    C:\Users\rpaul\Anaconda3\lib\site-packages\ipykernel\__main__.py:22: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead
    
    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/indexing.html#indexing-view-versus-copy
    




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
    </tr>
    <tr>
      <th>Spending Ranges(Per Student)</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>&lt;585</th>
      <td>83.363065</td>
      <td>83.964039</td>
      <td>93.702889</td>
      <td>96.686558</td>
      <td>95.194724</td>
    </tr>
    <tr>
      <th>$585-615</th>
      <td>83.529196</td>
      <td>83.838414</td>
      <td>94.124128</td>
      <td>95.886889</td>
      <td>95.005509</td>
    </tr>
    <tr>
      <th>$615-645</th>
      <td>83.386723</td>
      <td>83.833709</td>
      <td>93.329036</td>
      <td>97.228489</td>
      <td>95.278762</td>
    </tr>
    <tr>
      <th>$645-675</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>



# Scores by School Spending for District schools


```python
# Create a table that breaks down school performances based on average Spending Ranges (Per Student), only for District schools

# Create the bins in which Data will be held
# Bins are 575 to 600, 600 to 625, 625 to 650, 650 to 675
bins = [0, 585, 615, 645, 675]

# Create the names for the four bins
group_names = ['<585', '$585-615', '$615-645', '$645-675']

#create a merged df of school and students
score_school_spend_df = schools_df[schools_df['type'] == "District"]
score_school_spend_df.rename(columns = {'name':'School Name',
                                    'size':'Total Students',
                                    'budget':'Total School Budget'}, inplace = True)
score_school_spend_df["Per Student Budget"] = (score_school_spend_df['Total School Budget'] / score_school_spend_df['Total Students'])
score_students_df = students_df[['school','Student ID','math_score','reading_score']]
score_students_df.rename(columns={'school':'School Name'}, inplace=True)
merged_score_school_spend_df = pd.merge(score_school_spend_df,score_students_df, on = 'School Name')
summary_score_school_spend_df = merged_score_school_spend_df[['School Name','Per Student Budget','Student ID','math_score','reading_score']]

# Cut postTestScore and place the scores into bins
summary_score_school_spend_df['Spending Ranges(Per Student)']=pd.cut(summary_score_school_spend_df['Per Student Budget'], bins, labels=group_names)

grouped_students_by_score = summary_score_school_spend_df.groupby('Spending Ranges(Per Student)')

score_math_avg = grouped_students_by_score['math_score'].mean()
score_reading_avg = grouped_students_by_score['reading_score'].mean()
score_total_students = grouped_students_by_score['Student ID'].size()

pass_math_df = summary_score_school_spend_df[summary_score_school_spend_df["math_score"] >= 70]
pass_math_by_score_df = pass_math_df.groupby('Spending Ranges(Per Student)')['math_score'].size()
pass_reading_df =  summary_score_school_spend_df[summary_score_school_spend_df["reading_score"] >= 70]
pass_reading_by_score_df =  pass_reading_df.groupby('Spending Ranges(Per Student)')['reading_score'].size()


#create new dataframe with calculated values
score_summary_table = pd.DataFrame({'Average Math Score': score_math_avg,
                                    'Average Reading Score': score_reading_avg,
                                   'Total Students': score_total_students,
                                   'Math_pass_count': pass_math_by_score_df,
                                   'Reading_pass_count': pass_reading_by_score_df})
score_summary_table['% Passing Math'] = (score_summary_table['Math_pass_count']/score_summary_table['Total Students'])*100
score_summary_table['% Passing Reading'] = (score_summary_table['Reading_pass_count']/score_summary_table['Total Students'])*100
score_summary_table['Overall Passing Rate'] = (score_summary_table['% Passing Math'] + score_summary_table['% Passing Reading']) / 2

#drop columns not needed
score_summary_table = score_summary_table.drop(score_summary_table.columns[[2,3,4]], axis=1)

#Display
score_summary_table
```

    C:\Users\rpaul\Anaconda3\lib\site-packages\pandas\core\frame.py:3027: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame
    
    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/indexing.html#indexing-view-versus-copy
      return super(DataFrame, self).rename(**kwargs)
    C:\Users\rpaul\Anaconda3\lib\site-packages\ipykernel\__main__.py:15: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead
    
    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/indexing.html#indexing-view-versus-copy
    C:\Users\rpaul\Anaconda3\lib\site-packages\ipykernel\__main__.py:22: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead
    
    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/indexing.html#indexing-view-versus-copy
    




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
    </tr>
    <tr>
      <th>Spending Ranges(Per Student)</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>&lt;585</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>$585-615</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>$615-645</th>
      <td>76.934734</td>
      <td>80.926277</td>
      <td>66.759872</td>
      <td>80.733820</td>
      <td>73.746846</td>
    </tr>
    <tr>
      <th>$645-675</th>
      <td>77.049297</td>
      <td>81.005604</td>
      <td>66.230813</td>
      <td>81.109397</td>
      <td>73.670105</td>
    </tr>
  </tbody>
</table>
</div>


