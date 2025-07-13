
# ClimAdvisor: Analyzing Forecast Reliability Across France

Welcome to **ClimAdvisor**, a data-driven project aimed at evaluating **weather reliability** across various French cities. This project focuses on **exploratory weather analysis** and the **accuracy of 5-day weather forecasts** using real-world data. It complements the *France Adventure Planner* project but operates as a standalone analytical deep dive.

---

## 🌤️ Project Objectives

- Analyze weather trends (temperature, humidity, rain probability, weather score) for key French cities.
- Compare **forecasted weather** against **real observed data** over a 5-day window.
- Identify **the most and least reliable** cities in terms of forecast accuracy.
- Provide a Power BI dashboard for visual exploration of accuracy metrics over time.
- Offer reusable code (Python, DAX) for reproducible and dynamic weather analytics.

---

## 🛠️ Tools and Technologies

- **Python**: Data collection, cleaning, transformation, and calculations.
  - `pandas`, `matplotlib`, `seaborn`
- **Power BI**: Visual storytelling, DAX for dynamic calculations
- **DAX**: Used in Power BI to compute weather indicators over time ranges
- **Jupyter Notebooks**: For exploratory data analysis
- **VS Code**: Python script development
- **GitHub**: Version control and project publishing

---

## 🌐 Data Sources

- **OpenWeatherMap API**: 5-day forecasts & current weather (temperature, humidity, weather condition, rain probability)
- **CSV files**: Cleaned and processed weather data, separated by real values and forecasted values for each day (`Day1`, `Day2`, ..., `Day5`)


[Explore the raw forecast data used in this project](https://github.com/JBaptisteAll/France_Adventure_Planner/tree/main/forecasts)

![Data Flow](https://github.com/JBaptisteAll/ClimAdvisor_Analyzing_Forecast_Reliability_Across_France/blob/main/Assets/Dataflow.png)
---

## 🔎 Exploratory Weather Analysis

We computed the **average temperature**, **humidity**, **rain probability**, and **custom weather scores** for each city, for each day (1 to 5). Below is an example of how average temperatures were visualized for the city of Brest:

```python
# Définir les bornes 
min_RainProb = all_data["Rain_Probability_jour1"].min()
max_RainProb = all_data["Rain_Probability_jour1"].max()

fig = px.density_mapbox(
    all_data,
    lat="Latitude_jour1",
    lon="Longitude_jour1",
    z="Rain_Probability_jour1",
    mapbox_style="open-street-map",
    animation_frame="Date_jour1",  # Animation basée sur la date
    zoom=3.5,
    radius=7,
    color_continuous_scale="YlGnBu",
    center={"lat": 46.603354, "lon": 1.888334},
    range_color=[min_RainProb, max_RainProb]  # Échelle de couleur fixe
)

fig.show()
```

**Output:**

![Rain_Probability](Assets\Rain_Prob_gif.gif)

---
### Evolution of Average Weather Indicators (National Level)

The following Python code was used to calculate and visualize the national daily average of four main weather indicators: temperature, weather score, rain probability, and humidity.

```python
# 1. Moyennes par jour pour chaque indicateur
Temp_Avg = {
    "Jour 1": all_data["Temp_Avg_jour1"].mean(),
    "Jour 2": all_data["Temp_Avg_jour2"].mean(),
    "Jour 3": all_data["Temp_Avg_jour3"].mean(),
    "Jour 4": all_data["Temp_Avg_jour4"].mean(),
    "Jour 5": all_data["Temp_Avg_jour5"].mean(),
}

Score_Meteo = {
    "Jour 1": all_data["Weather_Score_jour1"].mean(),
    "Jour 2": all_data["Weather_Score_jour2"].mean(),
    "Jour 3": all_data["Weather_Score_jour3"].mean(),
    "Jour 4": all_data["Weather_Score_jour4"].mean(),
    "Jour 5": all_data["Weather_Score_jour5"].mean(),
}

Rain_Prob = {
    "Jour 1": all_data["Rain_Probability_jour1"].mean(),
    "Jour 2": all_data["Rain_Probability_jour2"].mean(),
    "Jour 3": all_data["Rain_Probability_jour3"].mean(),
    "Jour 4": all_data["Rain_Probability_jour4"].mean(),
    "Jour 5": all_data["Rain_Probability_jour5"].mean(),
}

Humidity = {
    "Jour 1": all_data["Humidity_jour1"].mean(),
    "Jour 2": all_data["Humidity_jour2"].mean(),
    "Jour 3": all_data["Humidity_jour3"].mean(),
    "Jour 4": all_data["Humidity_jour4"].mean(),
    "Jour 5": all_data["Humidity_jour5"].mean(),
}

# 2. Créer les subplots
fig, axs = plt.subplots(2, 2, figsize=(14, 8))
fig.suptitle("Évolution des indicateurs météo", fontsize=16)

# Température
axs[0, 0].plot(list(Temp_Avg.keys()), list(Temp_Avg.values()), marker="o")
axs[0, 0].set_title("Température moyenne")
axs[0, 0].set_ylabel("°C")
axs[0, 0].grid(True)

# Score météo
axs[0, 1].plot(list(Score_Meteo.keys()), list(Score_Meteo.values()), marker="o", color="green")
axs[0, 1].set_title("Weather Score")
axs[0, 1].grid(True)

# Probabilité de pluie
axs[1, 0].plot(list(Rain_Prob.keys()), list(Rain_Prob.values()), marker="o", color="blue")
axs[1, 0].set_title("Probabilité de pluie (%)")
axs[1, 0].grid(True)

# Humidité
axs[1, 1].plot(list(Humidity.keys()), list(Humidity.values()), marker="o", color="purple")
axs[1, 1].set_title("Humidité (%)")
axs[1, 1].grid(True)

# 3. Ajuster l'espacement
plt.tight_layout(rect=[0, 0, 1, 0.95])
plt.show()
```
Output:
![Rain_Probability](Assets\Weather_indicator.png)

This chart highlights how significantly weather indicators can fluctuate within just a 
few days. For instance, even over a 4- to 5-day range, we notice that the average 
temperature and humidity can vary sharply, with gaps that would have seemed abnormal only
a decade ago.

Such volatility—especially when observed on a national average—raises concerns about 
climate instability. These variations are no longer local anomalies but increasingly 
reflect larger climatic trends that meteorologists and climate scientists have been 
warning about.

If such temperature shifts are possible in just a few days, it invites us to reflect on 
the forecasts for the coming decades. Perhaps our models—and our optimism—underestimate 
the pace and magnitude of climate change.

While this project focuses on forecast reliability, it also reminds us of the growing 
unpredictability of the climate itself.

---

## ✅ Forecast Accuracy Calculation

To determine **forecast reliability**, we compared the forecasted values from each day (`Day1`, `Day2`, etc.). We calculated absolute and relative errors for each indicator.

### Sample Python Logic

```python
# Liste des jours à comparer avec la référence (jour 1)
jours_previsions = ["jour2", "jour3", "jour4", "jour5"]

# Calcul des écarts relatifs pour chaque jour de prévision par rapport à jour1
for jour in jours_previsions:
    all_data[f"Temp_Error_{jour}"] = abs(all_data[f"Temp_Avg_{jour}"] - all_data["Temp_Avg_jour1"]) / all_data["Temp_Avg_jour1"]
    all_data[f"Rain_Error_{jour}"] = abs(all_data[f"Rain_Probability_{jour}"] - all_data["Rain_Probability_jour1"])
    all_data[f"Weather_Score_Error_{jour}"] = abs(all_data[f"Weather_Score_{jour}"] - all_data["Weather_Score_jour1"]) / 600

# Score d'accuracy basé sur les écarts
for jour in jours_previsions:
    all_data[f"Accuracy_{jour}"] = 100 - (
        (all_data[f"Temp_Error_{jour}"] * 40) + 
        (all_data[f"Rain_Error_{jour}"] * 30) + 
        (all_data[f"Weather_Score_Error_{jour}"] * 30)
    )

# Normalisation du score pour qu'il reste entre 0 et 100
for jour in jours_previsions:
    all_data[f"Accuracy_{jour}"] = all_data[f"Accuracy_{jour}"].clip(0, 100)

# regrouper pour chaque ville
city_accuracy = all_data.groupby("Ville_jour1")[[f"Accuracy_{jour}" for jour in jours_previsions]].mean()

# Afficher les résultats
display(city_accuracy)
```
Then visualized on a map, separating the dataset in half.
```python
# Visualiser les 10 meilleures et 10 pires villes sur une carte
top_25_cities = city_accuracy.head(25).copy()
#middle_50_cities = city_accuracy.iloc[20:-20].copy()
worst_25_cities = city_accuracy.tail(25).copy()

# Ajouter une colonne pour différencier les catégories
top_25_cities["Category"] = "Top 25"
worst_25_cities["Category"] = "Worst 25"

# Fusionner les deux catégories
top_worst_cities = pd.concat([top_25_cities,  worst_25_cities])

# Appliquer une taille uniforme pour toutes les villes
top_worst_cities["Size"] = 5  # Taille fixe pour tous les points

# Définir une palette de couleurs : Rouge pour le Top 10, Bleu pour le Worst 10
color_map = {"Top 25": "red", "Worst 25": "blue"}

# Carte interactive des 10 meilleures et 10 pires villes
fig = px.scatter_mapbox(
    top_worst_cities,
    lat="Latitude_jour1",
    lon="Longitude_jour1",
    color="Category",  
    size="Size",  
    hover_name="Ville_jour1",
    hover_data={"Mean_Accuracy": True},
    mapbox_style="open-street-map",
    zoom=3.5,
    color_discrete_map=color_map,
    size_max=10  
)

fig.show()
```
**Output:** 
Below is a comparison of the 25 most and least accurate cities regarding weather forecasts, plotted on an interactive map.

![Rain_Probability](Assets\newplot.png)
---

## 📊 Power BI Dashboard – Visual Examples

You can explore weather metrics, errors, and city comparisons interactively through our Power BI dashboard:

| Visual | Screenshot |
|--------|------------|
| **Dashboard** | ![Dashboard](Assets\Dashboard.png) |
| **France Overview** | ![Overview](Assets\Overview.png) |
| **Prediction Vs Reality** | ![Expected](Assets\PredictVsReal.png) |
| **Forecast Comparison** | ![Accuracy](Assets\Compare.png) |

To compare city or event time period between one and another.

---

## ⚙️ DAX Formula Examples (Power BI)

Below is the DAX formula used to dynamically calculate the average Weather Score over time, filtered by city and date range:

```DAX
Weather Score A = 
VAR SelectedDateMin = MIN(Dates_A[Date])
VAR SelectedDateMax = MAX(Dates_A[Date])
RETURN
IF(
    ISFILTERED(Ville_A[Ville]),
    CALCULATE(
        AVERAGE(Day1[Weather_Score]),
        Day1[Ville] = SELECTEDVALUE(Ville_A[Ville]),
        Day1[Date] >= SelectedDateMin,
        Day1[Date] <= SelectedDateMax
    ),
    CALCULATE(
        AVERAGE(Day1[Weather_Score]),
        Day1[Date] >= SelectedDateMin,
        Day1[Date] <= SelectedDateMax
    )
)
```

Similar measures were created for:
- `Rain_Prob A`
- `Avg_Temp A`


These calculations allow fully dynamic visuals when filters are applied to the dashboard.

---

## 🧠 What I Learned

- Learned how to connect and synchronize multiple tools such as **Python Notebooks** and **Power BI** with **GitHub** for a seamless workflow.
- Designed and implemented a multi-metric comparison framework to evaluate the accuracy of weather forecasts over time.
- Deepened my understanding of **DAX** and **Power BI**, using them to create interactive and dynamic dashboards with time and city filters.
- Strengthened my skills in **data cleaning**, visual analysis in **Python**, and effective **dashboard storytelling**.
- Gained experience in **managing a complete end-to-end project**, from building a user-facing app (France Adventure Planner) to reusing the same data for exploratory analysis and insight generation, adding more value to the original dataset.

---

## 🚧 Challenges Faced

- **Data alignment between forecasted values**: One of the main challenges was ensuring 
that forecast data (from Day 1 to Day 5) correctly aligned.
- **Power BI interactivity with filters**: Ensuring that DAX measures behaved correctly 
when multiple filters (city, date range) were applied.

- **Handling daily weather data with multiple sources**: Working with 5-day forecasts, 
real data, and multiple indicators across dozens of cities required strict organization 
and careful naming to avoid mixing files or data points.

- **Balancing automation and control**: Automating the entire pipeline (with GitHub 
Actions) was useful, but I had to double-check a lot of things manually to ensure clean 
data, especially with edge cases or sudden API gaps.

- **Designing something meaningful, not just functional**: I wanted this project to go 
beyond “data for data’s sake” — it had to be beautiful, useful, and thought-provoking. 
That required extra work in storytelling, dashboard design, and testing what visuals made 
sense.


---

## ✅ Conclusion

**ClimAdvisor** is a comprehensive analytical project that explores forecast accuracy using key weather indicators across French cities. By combining **Python analysis** with interactive **Power BI dashboards**, it offers both granular data validation and engaging visual storytelling. This project highlights practical **data analysis skills** and can easily be extended to include more cities, additional indicators, or historical trends.

---

*Designed by JB as part of a personal portfolio to showcase Python, Power BI, and DAX expertise — Let the weather guide your next journey!*
