---
uid: Working_with_behavioral_anomaly_detection
---

# Working with behavioral anomaly detection

> [!NOTE]
>
> - This feature requires [Storage as a Service](xref:STaaS) or a [self-hosted Cassandra-compatible database](xref:Supported_system_data_storage_architectures).
> - Anomaly detection is only available for numeric parameters that are not part of [partial tables](xref:Table_parameters#partial-tables). It is also limited to at most 100,000 parameters per DMA.
> - Anomaly detection is not available for discrete parameters. <!-- RN 35465 -->
> - You can enable or disable this feature via *System Center* > *System settings* > *analytics config*. However, note that disabling the feature will influence the trend graph insights generated by [relation learning](xref:Working_with_relation_learning).

> [!TIP]
> See also:
>
> - [Behavioral anomaly detection](xref:Behavioral_anomaly_detection)
> - [Intelligent Fault Detection](https://community.dataminer.services/video/intelligent-fault-detection-in-action) on DataMiner Dojo ![Video](~/user-guide/images/video_Duo.png)

From DataMiner 10.0.0/10.0.2 onwards, the DataMiner Analytics software can detect the changes in the behavior of a trend, also known as "change points". The following kinds of change points can be detected:

- **Flatline**: A fluctuating value suddenly remains constant. This type of change point can be detected from DataMiner 10.2.5/10.3.0 onwards.

- **Level shift**: A value shifts upwards or downwards and then stays at that level, e.g. a value fluctuating around 0 that starts to fluctuate around 10.

- **Outlier**: A value suddenly spikes upwards or downwards, but returns to its previous, normal behavior after a few points.

- **Trend change**: A value suddenly starts to increase or decrease at an unusual rate. For example, a value fluctuating around 10 (i.e. a trend slope of 0) that suddenly starts to increase by 1 unit per second (i.e. a trend slope of 1).

- **Variance change**: The variance of a value either increases or decreases. For example, a series like 0.5, 0.6, -0.5, -0.2, 1, …, 5, 8, 9, -5, -6, -2.1, … indicates a variance increase. The value is first fluctuating around 0 between 1 and -1 and then starts fluctuating around 0 between 10 and -10.

- **Unlabeled change**: If a change point cannot be classified as one of the above-mentioned change points, it is considered an unlabeled change.

If a change point other than an outlier or unlabeled change is unexpected, it will be considered anomalous. Level shifts that have a different direction than previous recent jumps or that jump to a previously unseen level will typically be labeled “anomalous”. Similarly, trend or variance changes will be labeled “anomalous” when no earlier trend or variance changes in the same direction were detected during the last weeks. A flatline will be considered anomalous when no recent flatline change point of approximately the same length or longer is detected.

From DataMiner 10.3.8/10.4.0 onwards<!-- RN 36664 -->, a change can also be considered anomalous if it has been seen before in the historical behavior of the parameter but it does not fit in the usual periodic pattern.

> [!NOTE]
>
> - Whenever an anomalous change point is detected, a “suggestion event” is generated, which is cleared again two hours after its creation time or its last update time. You can view these suggestion events by creating a suggestion event tab in the Alarm Console. See [Adding and removing alarm tabs in the Alarm Console](xref:ChangingTheAlarmConsoleLayout#adding-and-removing-alarm-tabs-in-the-alarm-console).
> - You can configure alarm templates to have alarms generated instead of suggestion events, depending on the parameter and the type of anomaly. See [Configuring anomaly detection alarms for specific parameters](xref:Configuring_anomaly_detection_alarms).
> - If a very high number of behavioral change points are detected in a short period, detection of behavioral anomalies is temporarily disabled to avoid unreliable results. This is indicated in the SLAnalytics logging. Prior to DataMiner 10.2.3/10.3.0, a notification is also displayed in the Alarm Console, which disappears again 2 hours after the change point flood has been resolved.

## Trend icons on data pages

Based on the behavioral anomaly detection, the following icons can be displayed next to a trended parameter on a data page:

| Icon   | Description     |
|--------|-----------------|
| ![Trend graph icon](~/user-guide/images/StandardTrendGraphIcon.png) | Standard trend graph icon |
| ![Upward arrow icon](~/user-guide/images/ArrowRight60.png) | Upward arrow  |
| ![Downward arrow icon](~/user-guide/images/ArrowRight120.png)  | Downward arrow  |
| ![Flat arrow icon](~/user-guide/images/ArrowRight.png)  | Flat arrow  |
| ![Upward level shift icon](~/user-guide/images/LevelShiftIncrease.png) ![Red upward level shift icon](~/user-guide/images/LevelShiftIncreaseRed.png) | Upward level shift |
| ![Downward level shift icon](~/user-guide/images/LevelShiftDecrease.png) ![Red downward level shift icon](~/user-guide/images/LevelShiftDecreaseRed.png) | Downward level shift |
| ![Upward trend change icon](~/user-guide/images/ArrowTrendChangeUp.png) ![Red upward trend change](~/user-guide/images/ArrowTrendChangeUpRed.png) | Upward trend change |
| ![Downward trend change icon](~/user-guide/images/ArrowTrendChangeDown.png) ![Red downward trend change](~/user-guide/images/ArrowTrendChangeDownRed.png) | Downward trend change |
| ![Variance increase icon](~/user-guide/images/ArrowVarianceChangeUp.png) ![Red variance increase icon](~/user-guide/images/ArrowVarianceChangeUpRed.png) | Variance increase |
| ![Variance decrease icon](~/user-guide/images/ArrowVarianceChangeDown.png) ![Red variance decrease icon](~/user-guide/images/ArrowVarianceChangeDownRed.png) | Variance decrease |
| ![Upward outlier icon](~/user-guide/images/ArrowOutlierUp.png) ![Red upward outlier icon](~/user-guide/images/ArrowOutlierUpRed.png) | Upward outlier |
| ![Downward outlier icon](~/user-guide/images/ArrowOutlierDown.png) ![Red downward outlier icon](~/user-guide/images/ArrowOutlierDownRed.png) | Downward outlier |
| ![Flatline icon](~/user-guide/images/ArrowFlatline.png) ![Red flatline icon](~/user-guide/images/ArrowFlatlineRed.png) | Flatline |

Please note the following regarding these icons:

- If you hover the mouse pointer over these icons, a tooltip will display additional information.

- If a change point is considered anomalous, the icon is displayed in red.

- The selection of a particular icon is based on the trend data behavior and the change points that have occurred within a configurable time interval. By default, this time interval is set to 3600 seconds. Prior to DataMiner 10.0.9/10.1.0, this is specified in the *arrowWindowLength* parameter in the file *C:\\Skyline DataMiner\\Files\\SLAnalytics.config*. From DataMiner 10.0.9/10.1.0 onwards, the time interval can be configured via *System Center* > *System settings* > *analytics config*.

## Change points in trend graphs

On a trend graph, a change point is indicated by a bar below the graph. The length of the bar indicates the approximate time frame in which the change started, the height of the bar indicates the importance of the change, and the color of the bar indicates the severity.

When you hover the mouse pointer over a change point bar, a semi-transparent ribbon will be displayed over the entire height of the trend graph, showing more information about the change point.

Labels of change points of type “trend change” will indicate the level of increase or decrease in seconds, minutes, hours or days depending on the value. If, for example, the value increases by 0.01 per second (i.e. 0.6 per minute, 36 per hour or 864 per day), the label will show an increase of 36 per hour as it is the smallest amount greater than 1.
