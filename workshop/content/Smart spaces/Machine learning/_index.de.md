+++
title = "Maschinelles Lernen"
weight = 30
pre = "<b>c. </b>"
+++

## Einleitung
Am Ende dieses Kapitels sollte Ihre serverlose Anwendung Folgendes tun:

- Trainiert ein maschinelles Lernmodell, das neue Thermostatmeldungen in Rückschlüsse auf die Raumbelegung umsetzt
- Hostet ein Modell für maschinelles Lernen auf einem API-Endpunkt zur Verwendung

## **Konzepte für das Training eines maschinellen Lernmodells**
Data Science und maschinelles Lernen sind riesige Gebiete für sich. Es würde den Rahmen dieses Moduls bei weitem sprengen, die Grundlagen des Trainings von Machine Learning-Modellen zu vermitteln. Glücklicherweise wurde die Werkzeugkette zum Erstellen neuer Modelle so weit vereinfacht, dass wir diese Werkzeugkette verwenden können, um mit ML zu experimentieren, wenn wir nur genug über unsere Daten wissen.

In dieser Lösung versuchen Sie, einen einfachen Schwellenwert in der IoT Core Rules Engine zu ersetzen, der den vom Gerät gemeldeten Schallpegel auswertet und einen neuen booleschen Schlüsselwert namens *roomOccupancy* erzwingt. Sie wissen aus der Betrachtung der Schalldaten in den gemeldeten Nachrichten, dass die Werte niedrig sind, wenn es ruhig ist, und höher, wenn es Lärm gibt. Daher wissen Sie, dass ein einfacher Schwellenwert wie &quot;größer als 10&quot; ein guter Ausgangspunkt für die Erzeugung des *roomOccupancy* Werts war. (In Ihrem speziellen Fall wäre ein anderer Schwellenwert für Umgebungsgeräusche und registrierte Aktivitäten vielleicht besser geeignet gewesen).

Sie wenden einen ähnlichen Ansatz an, indem Sie der Werkzeugkette für maschinelles Lernen eine Stichprobe von Thermostatdaten geben, die in dem Raum aufgezeichnet wurden, in dem Ihr Gerät eingesetzt wird, und dem Trainingsauftrag sagen: &quot;So sehen die Daten aus, wenn die *roomOccupancy* wahr sein sollte, und wie sie aussehen, wenn sie falsch ist.&quot; Der Trainingsjob wertet Ihren Datensatz aus, zielt auf die Spalte *roomOccupancy* und versucht, ein Modell zu erstellen, das die Werte von *roomOccupancy* basierend auf den Schallpegelbereichen und sogar den anderen Spalten wie Zeit, HLK-Status und Temperatur genau reproduziert.

Sobald Ihr erstes Modell trainiert ist, werden Sie im folgenden Kapitel diesen einfachen statischen Schwellenwert für den Schallpegel durch die abgeleitete Klassifizierung von _roomOccupancy ersetzen_, wie sie von Ihrem Modell ermittelt wurde!

## So richten Sie die serverlose Infrastruktur ein
Zunächst müssen Sie Amazon SageMaker Studio einrichten, um ein neues Experiment für die automatische Modellschulung zu konfigurieren.

1. Gehen Sie zur Amazon SageMaker-Konsole und wählen Sie **Amazon SageMaker Studio**.
2. Wählen Sie *Quick start* und geben Sie optional einen neuen _Benutzernamen_ ein.
3. Wählen Sie für _Ausführungsrolle_ das Dropdown-Menü und wählen Sie Neue IAM-Rolle erstellen.
4. Für _S3-Buckets_ geben Sie **None** an und wählen dann **Create role**. Die anderen Voreinstellungen für Buckets mit &quot;sagemaker&quot; im Namen sind für dieses Projekt ausreichend.
5. Wählen Sie **Submit**, um den Bereitstellungsprozess von SageMaker Studio zu starten. Dieser Schritt nimmt einige Minuten in Anspruch, um für Sie abgeschlossen zu werden.

Wenn Ihr SageMaker Studio die Bereitstellung abgeschlossen hat, ist der nächste Schritt, Ihr Studio zu öffnen und ein neues Projekt zu konfigurieren.

1. Wählen Sie in der Systemsteuerung von SageMaker Studio die Option **Open Studio**.
2. Wählen Sie in der Starterleiste die Option **New project**.
3. Wählen Sie unter *SageMaker project templates* die *MLOps template for model building, training, and deployment* wählen Sie dann **Select project template** aus.
4. Geben Sie Ihrem Projekt einen Namen und eine Beschreibung und wählen Sie dann **Create project**.

Sobald das Projekt erstellt ist, sehen Sie ein Projekt-Dashboard mit Registerkarten wie Repositories, Pipelines, Experimente usw. Lassen Sie diese Browser-Registerkarte zu SageMaker Studio geöffnet, damit Sie schnell zu dieser Seite zurückkehren können.

Die nächste Aufgabe besteht darin, zu AWS IoT Analytics zurückzukehren, damit Sie die aggregierten Thermostatdaten zur Verwendung durch Ihr neues ML-Projekt exportieren können.

1. Öffnen Sie die [AWS IoT Analytics-Konsole](https://us-west-2.console.aws.amazon.com/iotanalytics/home?region=us-west-2#/datasets) und wählen Sie Ihren Datensatz (angenommener Name ist `smartspace_dataset`).
2. Wählen Sie unter *Data set content delivery rules* die Option **Edit**.
3. Wählen Sie **Add rule** und wählen Sie dann **Deliver result to S3**.
4. Wählen Sie unter **S3 bucket**, **Please select a resource** und suchen Sie den S3-Bucket, der für Ihr SageMaker Studio-Projekt erstellt wurde. Er wird wie `sagemaker-project-p-somehashhere` benannt sein. Wenn es mehrere Buckets mit diesem Namen gibt, müssen Sie im SageMaker Studio-Projekt nach der zufälligen Hash-ID Ihres Projekts suchen. Sie können den Hash in anderen Ressourcen Ihres Projekts sehen, z. B. auf den Registerkarten &quot;Repositories&quot; und &quot;Pipelines&quot;.
5. Verwenden Sie unter *Bucket key expression* diesen Ausdruck: `data/smartspace/Version/!{iotanalytics:scheduleTime}_!{iotanalytics:versionId}.csv`
6. Wählen Sie unter *Role* die Option **Create new** (Neu erstellen) und geben Sie einen Namen für die IAM-Rolle an, die IoT Analytics Zugriff auf das Schreiben von Daten in Ihren S3-Bucket gewährt. Wählen Sie **Create role**.
7. Wählen Sie **Save**, um Ihre neue Lieferregel zu bestätigen.

{{< img "iot_analytics-dataset_delivery.en.png" "Content delivery rule" >}}

8. Um Ihren Datensatz zu generieren, der für die Schulung in Ihrem neuen Amazon S3-Bucket gespeichert wird, wählen Sie **Actions** und dann **Run now**. Sie sollten sehen, dass die _Ergebnisvorschau_ aktualisiert wird, wenn der Inhalt des Datensatzes generiert wurde.

{{< img "iot_analytics-dataset_run.en.png" "Running the data set" >}}
{{< img "iot_analytics-dataset_preview.en.png" "Preview of the data set" >}}

Sie sind nun bereit, Ihr ML-Experiment zurück in SageMaker Studio zu starten. Ein Experiment wird die gemeldeten Thermostatdaten, die gerade von Ihrem IoT Analytics-Datensatz exportiert wurden, als Eingaben verwenden. Sie werden das Experiment so konfigurieren, dass es nach Möglichkeiten sucht, die vorhandene Spalte &quot;roomOccupancy&quot; genau vorherzusagen. Der automatische Trainingsauftrag analysiert Ihre Daten auf relevante Algorithmen, die ausprobiert werden sollen, und führt dann 250 Trainingsaufträge mit unterschiedlichen Hyperparametern aus, wobei derjenige ausgewählt wird, der die beste Anpassung an Ihre Eingabetrainingsdaten bietet.

Bevor Sie Ihr ML-Experiment starten, sollten Sie mehrere Stunden an Daten von Ihrem Thermostat in dem Raum, den Sie analysieren möchten, gemeldet haben, und in dieser Zeit sollte der Raum eine Mischung aus aktiven und inaktiven Perioden gehabt haben. Ein automatisches ML-Experiment benötigt mindestens 500 Datenzeilen, um zu funktionieren, aber je mehr Daten Sie mitbringen, desto besser wird das Ergebnis sein. Wenn Sie noch mehr Daten generieren müssen, bevor Sie fortfahren, vergessen Sie nicht, den Datensatz in der IoT Analytics-Konsole erneut auszuführen (letzter Schritt der vorherigen Anleitungsliste), damit diese Ergebnisse für SageMaker in Ihrem Projekt-S3-Bucket verfügbar sind. Wenn Sie bereit sind, Ihr Experiment zu starten, lesen Sie weiter.

1. Kehren Sie zu Ihrem SageMaker Studio zurück, öffnen Sie Ihr Projekt, wählen Sie die Registerkarte *Experiments* und wählen Sie **Create autopilot experiment**.
2. Geben Sie Ihrem Experiment einen Namen.
3. Wählen Sie unter *Project* Ihr Projekt aus der Liste aus.
4. Suchen Sie unter *Connect your data* and *S3 bucket name* den S3-Bucket Ihres Projekts und wählen Sie ihn in der Liste aus. Dies ist derselbe, den Sie im vorherigen Kapitel für die Regel zur Bereitstellung von IoT-Analytics-Datensatzinhalten ausgewählt haben.
5. Suchen Sie unter *Dataset file name* Ihren IoT Analytics-Datensatzinhalt wie `data/smartspace/Version/1607276270943_3b4eb6bb-8533-4ac0-b8fd-1b62ac0020a2.csv`.
6. Wählen Sie unter *Target* die Raumbelegung.
7. Suchen Sie unter *Output data location* und *S3 bucket name* denselben Projekt-S3-Bucket in dieser Liste, den Sie in Schritt 4 ausgewählt haben, und wählen Sie ihn aus.
8. Geben Sie unter *Dataset directory name* `output/smartspace` ein und wählen Sie **Use input as S3 object key prefix "output/smartspace"**. Dies definiert ein neues Präfix im S3-Bucket, das für Ihre Ausgabedateien verwendet wird.
9. Wählen Sie **Create Experiment**, um das automatisierte ML-Experiment zu starten.

Die Ausführung des Experiments kann Minuten bis Stunden dauern. Sie können den Fortschritt des Experiments in der Browser-Registerkarte von SageMaker Studio mitverfolgen, aber es ist auch sicher, die Registerkarte zu schließen und später wiederzukommen, um den Fortschritt zu überprüfen.

Nach Abschluss des Experiments werden 250 Versuche ausgegeben, mit denen SageMaker die besten Parameter für den Abstimmjob gefunden hat. Sortieren Sie die Tabelle mit den Versuchen, um denjenigen zu finden, der als *Best* markiert ist. Der nächste Meilenstein ist die Bereitstellung dieses Versuchs als Modell-Endpunkt, damit Sie ihn als API aufrufen können.

1. Wählen Sie die mit _Best_ markierte Studie und wählen Sie **Deploy model**.
   {{< img "sagemaker-trials.png" "SageMaker trials" >}}
2. Geben Sie Ihrem Endpunkt einen Namen. Die weiteren Schritte in diesem Modul nehmen den Namen `roomOccupancyEndpoint` an.
3. Wählen Sie unter *Inference Response Content* sowohl *predicted_label* als auch *probability* aus. *predicted_label* kann bereits zur Liste hinzugefügt worden sein.
4. Wählen Sie **Deploy model**, um SageMaker anzuweisen, Ihr Modell als neuen verbrauchbaren API-Endpunkt bereitzustellen. Dies wird einige Minuten dauern.

{{< img "sagemaker-deploy.png" "SageMaker deploy" >}}

Jetzt wird Ihr Machine Learning-Modell als API-Endpunkt bereitgestellt, der von Amazon SageMaker verwaltet wird. Im nächsten Kapitel, **Arbeiten mit ML-Modellen**, werden Sie den API-Endpunkt mit einer serverlosen Funktion konsumieren und die einfache Schwellenwertlogik in der IoT-Kernregel, die den `roomOccupancy` Wert bestimmt, durch von Ihrem Modell generierte Schlussfolgerungen ersetzen.

{{% notice warning %}}
Wenn Sie diese Anwendung länger als 6 Stunden laufen lassen, können AWS-Gebühren aufgrund der Anzahl der Anforderungen an den S3-Bucket anfallen. Es wird empfohlen, dieses Lernprogramm in dieser Zeit zu beenden und die [Bereinigungsschritte](/de/smart-spaces/conclusion.html#clean-up) durchzuführen, um unerwünschte Kosten zu vermeiden.
{{% /notice %}}

## Validierungsschritte
Bevor Sie mit dem nächsten Kapitel fortfahren, können Sie überprüfen, ob Ihre serverlose Anwendung wie vorgesehen konfiguriert ist:

1. In der [Amazon SageMaker-Konsole](https://us-west-2.console.aws.amazon.com/sagemaker/home?region=us-west-2#/endpoints) sollten Sie Ihren neuen Endpunkt mit dem Status _InService_ auf der Seite _Endpunkte_ sehen können.

{{< img "sagemaker-endpoints.png" "SageMaker endpoints" >}}

Wenn diese wie erwartet funktionieren, lassen Sie uns mit dem [Arbeiten mit ML-Modellen](/de/smart-spaces/working-with-ml-models.html) fortfahren.

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://community.m5stack.com/category/41/core2-for-aws" icon="far fa-question-circle" %}}Community support{{% /button %}}