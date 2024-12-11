import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from flask import Flask, jsonify

# Simulating data for 10 yoga sessions
np.random.seed(42)
sessions = pd.DataFrame({
    "SessionID": range(1, 51),
    "PoseAccuracy": np.random.uniform(70, 100, 50),  # Accuracy in %
    "Duration": np.random.randint(20, 60, 50),       # Duration in minutes
    "Engagement": np.random.uniform(0.5, 1.0, 50)    # Engagement score (0 to 1)
})

sessions["Consistency"] = sessions["PoseAccuracy"] * sessions["Engagement"]
sessions.head()

plt.figure(figsize=(10, 6))
sns.barplot(data=sessions, x="SessionID", y="PoseAccuracy")
plt.title("Pose Accuracy Across Sessions")
plt.xlabel("Session ID")
plt.ylabel("Pose Accuracy (%)")
plt.show()

from sklearn.preprocessing import MinMaxScaler

scaler = MinMaxScaler()
scaled_data = scaler.fit_transform(sessions[["PoseAccuracy", "Duration", "Engagement"]])

from sklearn.cluster import KMeans

# Define a function to calculate the silhouette score
from sklearn.metrics import silhouette_score

def calculate_silhouette_score(data, cluster_labels):
    return silhouette_score(data, cluster_labels)

# Define a function to plot the clusters
def plot_clusters(data, cluster_labels):
    plt.figure(figsize=(10, 6))
    sns.scatterplot(data=data, x="PoseAccuracy", y="Engagement", hue=sessions["Cluster"],  palette="viridis",)
    plt.title("Clusters of Sessions")
    plt.xlabel("Pose Accuracy (%)")
    plt.ylabel("Engagement Score")
    plt.show()

# Scale the data
from sklearn.preprocessing import MinMaxScaler
scaler = MinMaxScaler()
scaled_data = scaler.fit_transform(sessions[["PoseAccuracy", "Duration", "Engagement"]])

# Perform K-means clustering
kmeans = KMeans(n_clusters=2, random_state=42)
sessions["Cluster"] = kmeans.fit_predict(scaled_data)

# Calculate the silhouette score
silhouette = calculate_silhouette_score(scaled_data, sessions["Cluster"])
print(f"Silhouette Score: {silhouette}")

# Plot the clusters
plot_clusters(sessions, sessions["Cluster"])

sessions.to_csv("yoga_sessions.csv", index=False)
print("Data saved to yoga_sessions.csv")

# Flask App for Serving Insights
app = Flask(__name__)

@app.route("/", methods=["GET"])
def home():
    return "Welcome to the Yoga Sessions Insights API! Access insights at /insights"

@app.route("/insights", methods=["GET"])
def insights():
    return jsonify(sessions.to_dict(orient="records"))

if __name__ == "__main__":
    app.run(debug=True)
