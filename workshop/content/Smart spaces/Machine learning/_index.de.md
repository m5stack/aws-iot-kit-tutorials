+++
title = "Maschinelles Lernen"
weight = 30
pre = "<b>c. </b>"
+++

## Einführung in das Kapitel
Am Ende dieses Kapitels sollte Ihre serverlose Anwendung Folgendes tun:

* Trainieren Sie ein Modell für maschinelles Lernen, das neue Thermostat-meldungen nutzt, um auf die Raumbelegung rückzuschließen.
* Hosten Sie ein Modell für maschinelles Lernen auf einem API-Endpunkt.

## Konzepte für das Training eines Modells für maschinelles Lernen
Datenwissenschaft und maschinelles Lernen sind für sich genommen enorme Bereiche. Es geht weit über den Rahmen dieses Moduls hinaus, die Grundlagen der Funktionsweise des Modelltrainings für maschinelles Lernen zu vermitteln. Glücklicherweise wurde die Werkzeugkette zum Erstellen neuer Modelle so vereinfacht, dass wir diese Werkzeugkette verwenden können, um mit ML zu experimentieren, sobald wir genug über unsere Daten wissen.

In dieser Lösung versuchen Sie, einen einfachen Schwellenwert in der IoT-Core Regel zu verwenden, der den gemeldeten Geräuschpegel auswertet und einen neuen booleschen Schlüsselwert namens *roomOccupancy* erzeugt. Sie wissen aus den Tondaten in den Nachrichten, dass die Werte niedrig sind, wenn es leise ist, und höher, wenn es Umgebungsgeräusche gibt. Daher wissen Sie, dass ein einfacher Schwellenwert wie „größer als 10“ ein guter Ausgangspunkt war, um den *roomOccupancy* -Wert zu generieren. (In Ihrem speziellen Fall war möglicherweise ein anderer Schwellenwert für Umgebungsgeräusche angemessener!)

Sie wenden einen ähnlichen Ansatz an, indem Sie der ML-Toolchain einen Auszug der Thermostat-Daten geben, die aus dem Raum aufgezeichnet wurden. So teilen Sie dem Trainingsjob mit, wie die Daten aussehen, wenn die Raumbelegung wahr sein sollte und wie sie aussehen, wenn sie falsch ist. Der Trainingsjob wertet Ihren Datensatz aus, zielt auf die Spalte *roomOccupancy* ab und versucht, ein Modell zu erstellen, das die *roomOccupancy*-Werte basierend auf den Schallpegelbereichen und sogar den anderen Spalten wie Zeit, HLK-Status und Temperatur genau wiedergibt.

Sobald Ihr erstes Modell trainiert ist, ersetzen Sie im folgenden Kapitel diesen einfachen statischen Schwellenwert durch die abgeleitete Klassifizierung von *roomOccupancy*, die von Ihrem Modell bestimmt wurde!

## So richten Sie die Infrastruktur ohne Server ein
Zunächst richten Sie Amazon SageMaker Studio ein, um ein neues Experiment für das automatische Modelltraining zu konfigurieren.

1. Gehen Sie zur Amazon SageMaker-Konsole und wählen Sie **Amazon SageMaker Studio**.
2. Wählen Sie *Schnelleinrichtung* und geben Sie optional einen neuen *Benutzernamen* ein.
3. Wählen Sie für *Ausführungsrolle* das Dropdown-Menü aus und wählen Sie **Neue Rolle erstellen**.
4. Für *S3-Buckets*, wählen Sie **Keine** und dann **Rolle erstellen**. Die anderen Standardeinstellungen für Buckets mit „Sagemaker“ im Namen sind für dieses Projekt ausreichend.
5. Wählen Sie **Absenden**, um den Bereitstellungsprozess von SageMaker Studio zu starten. Dieser Schritt dauert einige Minuten, bis er abgeschlossen ist.

Sobald Ihr SageMaker Studio die Bereitstellung abgeschlossen hat, müssen Sie im nächsten Schritt Ihr Studio öffnen und ein neues Projekt konfigurieren.

1. Wählen Sie in der SageMaker Studio-Systemsteuerung **Studio öffnen**.
2. Wählen Sie auf der Registerkarte Launcher **Neues Projekt**.
3. Wählen Sie unter *SageMaker-Templates* *MLOps template for model building, training, and deployment* aus und wählen Sie dann **Select project template**.
4. Gib deinem Projekt einen Namen und eine Beschreibung und wähle dann **Projekt erstellen**.

Sobald das Projekt erstellt wurde, wird ein Projekt-Dashboard mit Registerkarten wie Repositories, Pipelines, Experimenten usw. angezeigt. Lassen Sie diesen Browser-Tab für SageMaker Studio geöffnet, damit Sie schnell zu dieser Seite zurückkehren können.

Die nächste Aufgabe besteht darin, zu AWS IoT Analytics zurückzukehren, damit Sie die aggregierten Thermostatdaten zur Verwendung in Ihrem neuen ML-Projekt exportieren können.

1. Öffnen Sie die [AWS IoT Analytics-Konsole](https://us-west-2.console.aws.amazon.com/iotanalytics/home?region=us-west-2#/datasets) und wählen Sie Ihren Datensatz (wir nehmen an, der Name ist `smartspace_dataset`).
2. Wählen Sie unter *Regeln für die Bereitstellung vom Datensatz* **Bearbeiten**.
3. Wählen Sie **Regel hinzufügen** und wählen Sie dann **Ergebnis an S3 liefern**.
4. Wählen Sie unter **S3-Bucket** **Bitte wählen Sie eine Ressource** und suchen Sie den S3-Bucket, der für Ihr SageMaker Studio-Projekt erstellt wurde. Es wird wie `sagemaker-project-p-somehashhere` benannt. Wenn es mehrere Buckets mit diesem Namen gibt, müssen Sie das SageMaker Studio-Projekt auf die zufällige Hash-ID Ihres Projekts überprüfen. Sie können den Hash in anderen Ressourcen Ihres Projekts wie den Registerkarten Repositories und Pipelines sehen.
5. Verwenden Sie unter *Bucket Key Expression* diesen Ausdruck:  `data/smartspace/Version/!{iotanalytics:scheduleTime}_!{iotanalytics:versionId}.csv`
6. Wählen Sie unter *Rolle* **Neu erstellen** und geben Sie einen Namen für die IAM-Rolle an, die IoT Analytics Zugriff zum Schreiben von Daten in Ihren S3-Bucket gewährt. Wählen Sie **Rolle erstellen**.
7. Wählen Sie **Speichern**, um Ihre neue Lieferregel abzuschließen.
{{< img "iot_analytics-dataset_delivery.en.png" "Content delivery rule" >}}

8. Um einen Datensatz zu generieren, der für das Training in Ihrem neuen Amazon S3-Bucket gespeichert wird, wählen Sie **Aktionen** und dann **Jetzt ausführen**. Sie sollten das Update *Ergebnisvorschau* sehen, wenn der Datensatzinhalt generiert wurde.

{{< img "iot_analytics-dataset_run.en.png" "Running the data set""1 - Actions, 2 - Run now" >}}
{{< img "iot_analytics-dataset_preview.en.png" "Preview of the data set" >}}

Sie können jetzt Ihr ML-Experiment wieder in SageMaker Studio starten. Bei einem Experiment werden die gemeldeten Thermostatdaten, die gerade von Ihrem IoT Analytics-Datensatz exportiert wurden, als Eingabe verwendet. Sie werden das Experiment so konfigurieren, dass nach Möglichkeiten gesucht wird, die vorhandene Spalte Raumbelegung genau vorherzusagen. Der automatische Trainingsjob analysiert Ihre Daten auf relevante Algorithmen, die Sie ausprobieren können. Dann führt er 250 Trainingsjobs mit unterschiedlichen Hyperparametern aus, wobei Sie diejenigen auswählen, die am besten zu Ihren eingegebenen Trainingsdaten passen.

Bevor Sie mit dem ML-Experiment beginnen, sollten Sie mehrere Stunden lang Daten in dem entsprechenden Raum gesammelt haben. In dieser Zeit sollte der Raum eine Mischung aus aktiven und inaktiven Perioden aufweisen. Ein automatisches ML-Experiment benötigt mindestens 500 Datenzeilen, um zu funktionieren. Je mehr Daten Sie einbringen, desto besser ist das Ergebnis. Wenn Sie noch weitere Daten generieren müssen, bevor Sie fortfahren, vergessen Sie nicht, den Datensatz in der IoT Analytics-Konsole erneut auszuführen (letzter Schritt der vorherigen Befehlsliste), damit diese Ergebnisse für SageMaker in Ihrem Projekt-S3-Bucket verfügbar sind. Wenn Sie bereit sind, Ihr Experiment zu beginnen, lesen Sie weiter.

1. Kehren Sie zu Ihrem SageMaker Studio zurück, öffnen Sie Ihr Projekt, wählen Sie den Tab *Experiments* und wählen Sie **Autopilot-Experiment erstellen**.
2. Geben Sie Ihrem Experiment einen Namen.
3. Wählen Sie unter *Projekt* Ihr Projekt aus der Liste aus.
4. Suchen Sie unter *Verbinden Sie Ihre Daten* und *S3-Bucket-Name* den S3-Bucket Ihres Projekts in der Liste und wählen Sie ihn aus. Dies ist dieselbe, die Sie im vorherigen Kapitel für die Inhaltsbereitstellungsregel für IoT Analytics-Datensätze ausgewählt haben.
5. Unter *Datensatzdateiname* suchen und wählen Sie den Inhalt Ihres IoT Analytics-Datensatzes wie `Data/SmartSpace/Version/1607276270943_3b4eb6bb-8533-4ac0-B8fd-1b62ac0020a2.csv`.
6. Unter *Ziel* wählen Sie `roomoccupancy`.
7. Suchen und wählen Sie unter *Output data location* und *S3 Bucket name* denselben Projekt-S3-Bucket in dieser Liste aus, den Sie in Schritt 4 ausgewählt haben.
8. Unter *Datensatzverzeichnisname* geben Sie `output/smartspace` ein und wählen Sie **Eingabe als S3-Objektschlüssel-Präfix „output/smartspace" verwenden**. Dadurch wird ein neuer Präfix im S3-Bucket definiert, der für Ihre Ausgabedateien verwendet wird.
9. Wählen Sie **Experiment erstellen**, um das automatisierte ML-Experiment zu starten.

Die Durchführung des Experiments kann Minuten bis Stunden dauern. Sie können den Fortschritt des Experiments auf der SageMaker Studio-Browserregisterkarte verfolgen, aber es ist auch möglich, die Registerkarte zu schließen und später wiederzukommen, um den Fortschritt zu überprüfen.

Sobald das Experiment abgeschlossen ist, ergibt sich die resultierende Ausgabe aus 250 Versuchen, die SageMaker verwendet hat, um die besten Parameter für Tuning-Jobs zu finden. Sortieren Sie die Tabelle der Versuche, um die mit *Best* markierte zu finden. Der nächste Meilenstein ist die Bereitstellung dieser Testversion als Modellendpunkt, damit Sie sie als API aufrufen können.

1. Wählen Sie die mit *Best* markierte Testversion aus und wählen Sie **Deploy model**. 
{{< img "sagemaker-trials.png" "SageMaker trials" "1 - Best, 2 - Deploy model" >}}
2. Geben Sie Ihrem Endpunkt einen Namen. Weitere Schritte in diesem Modul gehen von dem Namen `RoomOccupancyEndPoint` aus.
3. Wählen Sie unter *Inference Response Content* sowohl *predicted_label* als auch *probability* aus. *predicted_label* wurde möglicherweise bereits zur Liste hinzugefügt.
4. Wählen Sie **Deploy model**, um SageMaker anzuweisen, Ihr Modell als neuen API-Endpunkt bereitzustellen. Das dauert einige Minuten.
   {{< img "sagemaker-deploy.png" "SageMaker deploy" >}}
   
Jetzt wird Ihr Modell für maschinelles Lernen als API-Endpunkt bereitgestellt, der von Amazon SageMaker verwaltet wird. Im nächsten Kapitel, **Arbeiten mit ML-Modellen**, verwenden Sie den API-Endpunkt mit einer serverlosen Funktion und ersetzen die einfache Schwellenwertlogik in der IoT-Core-Regel durch von Ihrem Modell generierte Rückschlüsse.

{{% notice warning %}}
Wenn diese Anwendung länger als 6 Stunden ausgeführt wird, kann dies zu AWS-Gebühren aufgrund der Anzahl der Anfragen an den S3-Bucket führen. Wir empfehlen Ihnen, dieses Tutorial nach dieser Zeitspanne zu beenden und die [Bereinigungsschritte](/de/smart-spaces/conclusion.html#bereiningen) durchzuführen, um unerwünschte Kosten zu vermeiden.
{{% /notice %}}

## Schritte zur Validierung
Bevor Sie mit dem nächsten Kapitel fortfahren, können Sie überprüfen, ob Ihre serverlose Anwendung wie beabsichtigt konfiguriert ist:
1. Verwenden Sie die [Amazon SageMaker-Konsole](https://us-west-2.console.aws.amazon.com/sagemaker/home?region=us-west-2#/endpoints), um Ihren neuen Endpoint mit dem Status *InService* auf der Seite *Endpoints* zu sehen.
{{< img "sagemaker-endpoints.png" "SageMaker endpoints" >}}

Wenn dies wie erwartet funktionieren, gehen wir zu [**Mit ML-Modellen arbeiten**](/de/smart-spaces/working-with-ml-models.html) über.

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}