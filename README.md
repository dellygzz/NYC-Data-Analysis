
import pandas as pd

baseURL = 'data.cityofnewyork.us'
appToken = 'XM9Q07hq0LRv5aTTpgJ0aeXyD'
time = 60

dataset = '8586-3zfm'
chunk = 1000

restInspection = Socrata(baseURL, appToken, timeout = time)

# Load dataset

df = pd.read_csv("APUC24_Nov18.csv")

# Preview data
print("First few rows of the dataset:")
print(df.head())
# a. Remove duplicate rows
df = df.drop_duplicates()
print(f"Shape after removing duplicates: {df.shape}")

# Check for missing values
print("Missing values in each column:")
print(df.isnull().sum())

# Handle missing data
df = df.dropna(subset=['award'])  # Drop rows with missing critical values
df['consttype'] = df['consttype'].fillna('Unknown')  # Fill non-critical missing values
# b. Rename columns for clarity
df = df.rename(columns={
    'name': 'ProjectName',
    'boro': 'Borough',
    'geo_dist': 'GeographicDistrict',
    'projdesc': 'ProjectDescription',
    'award': 'AwardAmount',
    'consttype': 'ConstructionType',
    'buildingid': 'BuildingID',
    'building_address': 'BuildingAddress',
    'city': 'City',
    'zip_code': 'ZipCode',
    'community_board': 'CommunityBoard',
    'community_council': 'CommunityCouncil',
    'census_tract': 'CensusTract',
    'nta': 'NeighborhoodTabulationArea',
})

print("Columns after renaming:")
print(df.columns)
# c. Convert data types
df['AwardAmount'] = pd.to_numeric(df['AwardAmount'], errors='coerce')  # Convert award amounts to numeric
if 'ZipCode' in df.columns:
    df['ZipCode'] = df['ZipCode'].astype(str)  # Ensure zip codes are strings
	
# a. Generate summary statistics
print("Summary statistics of the dataset:")
print(df.describe(include='all'))
# b. Analysis by geographical units
def analyze_geographical_unit(column_name):
    if column_name in df.columns:
        print(f"\nAnalysis by {column_name}:")
        analysis = df.groupby(column_name)['AwardAmount'].agg(['sum', 'mean', 'count']).sort_values(by='sum', ascending=False)
        analysis['sum'] = analysis['sum']
        analysis['mean'] = analysis['mean']
        print(analysis.head(10))  # Show top 10 for simplicity

# Analyze by ZipCode
analyze_geographical_unit('ZipCode')

# Analyze by Borough
analyze_geographical_unit('Borough')

# Analyze by NeighborhoodTabulationArea
analyze_geographical_unit('NeighborhoodTabulationArea')

# Analyze by CommunityCouncil
analyze_geographical_unit('CommunityCouncil')
# c. Boroughs with highest and lowest award amounts
borough_awards = df.groupby('Borough')['AwardAmount'].agg(['sum', 'mean'])
borough_awards['sum'] = borough_awards['sum']
borough_awards['mean'] = borough_awards['mean']
print("\nTotal and Average Award Amounts by Borough:")
print(borough_awards.sort_values(by='sum', ascending=False))
	
# d. Generate analysis by different categorical values e.g., violation codes, job type, agency, etc.
import matplotlib
grouped_data = (df.groupby(['GeographicDistrict','Borough', 'ProjectName',]).agg(
    total_award=('AwardAmount', 'sum'),  
    average_award=('AwardAmount', 'mean'),
    project_count=('AwardAmount', 'size')).reset_index().sort_values(by='total_award', ascending=False))

print("\nGrouped Analysis by Borough and Geographical District:\n")
print(grouped_data.head(10))
#creating table for total award per borough and geo dis
total_award_by_borough = grouped_data.groupby(['Borough'])['total_award'].sum().reset_index()
print("\nTotal Award by Borough:")
(total_award_by_borough)
total_award_by_geodist = grouped_data.groupby('GeographicDistrict')['total_award'].sum().reset_index()
total_award_by_geodist = total_award_by_geodist.sort_values(by='total_award', ascending=False).head(10)
print("\nTotal Award by Geographical District:")

(total_award_by_geodist)

#creating bar graph for total award by boro and geo dist.

import matplotlib.pyplot as plt
total_award_by_borough.set_index('Borough', inplace=True)
total_award_by_borough.plot(kind= 'bar', title= "Total Award Amount per Borough")
total_award_by_geodist.set_index('GeographicDistrict', inplace=True)
total_award_by_geodist.plot( kind= 'bar', title= "Total Award Amount per District")
#making table for avg award by boro and geo district 
average_award_by_borough = grouped_data.groupby(['Borough'])['average_award'].mean().reset_index()
print("\nAverage Award by Borough:")
(average_award_by_borough)
average_award_by_geodist = grouped_data.groupby(['GeographicDistrict'])['average_award'].mean().reset_index()
average_award_by_geodist=average_award_by_geodist.sort_values(by='average_award', ascending=False).head(10)
print("\n Average Award by Geographical District:")
(average_award_by_geodist)
# bar graph for average award by borough and geographical district
average_award_by_borough.set_index('Borough', inplace=True)
(average_award_by_borough).plot(kind= 'bar', title= "Average Award per Borough")
average_award_by_geodist.set_index('GeographicDistrict', inplace=True)
(average_award_by_geodist).plot(kind= 'bar', title= "Average Award per Borough")
# e. Create at least one calculated column e.g., how long it takes to close a complaint, fix a violation, etc.
grouped_data = (df.groupby(['Borough']).agg(total_award=('AwardAmount', 'sum'),project_count=('ProjectName', 'size')).reset_index())

overall_total_award = grouped_data['total_award'].sum()
grouped_data['percent_of_total'] = (grouped_data['total_award'] / overall_total_award) * 100

print("\nGrouped Analysis by Borough:\n")
(grouped_data[['Borough', 'total_award', 'project_count', 'percent_of_total']])
#creating bar graph of percentages 
grouped_data.set_index('Borough', inplace=True)
grouped_data['percent_of_total'].plot(kind= 'bar', title= "Borough Total Award by Percentages")
















