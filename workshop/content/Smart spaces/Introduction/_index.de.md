+++
title = "Einführung"
weight = 10
pre = "<b>a. </b>"
+++

## Ihre Aufgabe

In diesem Szenario nehmen Sie die Rolle eines Full-Stack-Entwicklers ein, der mit der Automatisierung des Thermostats eines Besprechungsraums beauftragt ist, um Energie zu sparen. Jetzt untersuchen Sie, wie Sie den Mechanismus verbessern können, um festzustellen, ob der Raum belegt ist. Ein einfaches Modell für maschinelles Lernen könnte Ihnen helfen, eine intelligentere Klassifizierung als einen statischen numerischen Schwellenwert zu identifizieren. So können die Daten der Vergangenheit berücksichtigen werden, um Jitter oder Fehlalarme zu reduzieren.

Das Training eines Modells für maschinelles Lernen ist heutzutage mit der modernen Tool-Kette relativ einfach. Der Aufbau eines *guten* Modells ist jedoch schwierig und erfordert Fachwissen, ein umfassendes Verständnis der Inputs und Anstrengungen über viele Zyklen hinweg, um es zu optimieren. Heute probieren Sie die Amazon SageMaker-Toolkette aus, um zu erfahren, wie IoT-Daten zum Trainieren eines Modells verwendet werden können. Allerdings können wir nicht erwarten, dass gleich beim ersten Versuch ein *gutes* Modell erstellt wird.

Bitte beachten Sie, dass die Fertigstellung dieses Moduls ungefähr zwei Tage dauern wird. Während Sie nur eine Stunde praktische Arbeit zu erledigen haben, gibt es zwei Schritte, in denen Sie keine aktive Arbeit leisten müssen. Im ersten Schritt stellen Sie Ihr intelligentes Thermostat in dem Raum bereit, den Sie studieren möchten. Dann sammeln Sie Telemetrie-Daten, um sie für das ML-Training zu speichern. Sie sollten Ihr Gerät bereitstellen und Daten mindestens für einige Stunden (vorzugsweise 24 Stunden) sammeln, aber je mehr Daten Sie sammeln, desto genauer kann das ML-Modell sein. Der zweite Schritt ist der automatisierte Trainingsprozess des ML-Modells, nachdem Sie ausreichend Daten gesammelt haben. Dieser Trainingsprozess kann mehrere Stunden dauern und wir empfehlen, ihn morgens zu beginnen und am Nachmittag wieder zu ihm zurückzukehren oder ihn über Nacht laufen zu lassen.

## Probleme lösen

Die im letzten Modul entwickelte Lösung verwendet einen statischen numerischen Schwellenwert für den eingehenden Geräuschpegel, um einen neuen Schlüsselwert namens *roomOccupancy* zu erzeugen. Um ein einfaches Modell für maschinelles Lernen für eine ähnliche Funktion zu trainieren, verwenden Sie vorhandene Daten als Grundlage für das Training. Das bedeutet, dass Sie die vorhandene Lösung mehrere Stunden lang in einem Raum laufen lassen müssen, in dem der Raum abwechselnd belegt ist oder nicht. Sie verwenden einen aggregierten Datensatz dieser Geräte-Telemetrie, um ein automatisiertes ML-Trainingsexperiment zu ermöglichen, das dann verwendet wird, um neue Geräteberichte mit einem neuen *RoomOccupancy*-Wert zu klassifizieren. Auch hier ist Ihr erstes trainiertes Modell möglicherweise nicht sehr genau, aber der Zweck dieser Lösung besteht darin, Ihnen praktische Erfahrung mit dem Speichern von IoT-Daten, dem Ausführen des Trainingsablaufs und des Verbrauchs eines neuen ML-Modells zu vermitteln.

## Architektur der Lösung
![Smart Spaces-Architektur](introduction/smartspace-overview.png)

Der Workflow, den Sie in diesem Modul bereitstellen, umfasst die folgenden Schlüsselkomponenten:

1. Eine neue IoT-Core-Regel leitet Aktualisierungen des Geräteschattens an einen Dienst namens AWS IoT Analytics weiter. Der IoT-Analytics-Dienst speichert IoT-Rohdaten in großen Mengen, wandelt sie um und bereinigt sie, um Rohdaten in ein Datenformat umzuwandeln.
2. Ihr IoT Analytics-Projekt besteht aus vier miteinander verknüpften Ressourcen: Einem Kanal zum Speichern von IoT-Rohdaten aus dem Geräteschatten. Einer Pipeline zum Transformieren/Filtern/Anreichern von Daten. Einem Datenspeicher für verarbeitete Daten. Einem Datensatz, der gespeicherte Abfragen ausführt und die Ergebnisse zur Verarbeitung weitervermitteln kann.
3. Amazon SageMaker Studio ist eine integrierte Umgebung für maschinelles Lernen, in der Sie Ihre Modelle erstellen, trainieren, bereitstellen und analysieren können.
4. Ein Amazon SageMaker-Endpunkt, der Ihr trainiertes Modell als API hostet.
5. Eine AWS Lambda-Funktion, die einfachen Code ausführt, um Ihre vom Thermostat veröffentlichten Meldungen zu verarbeiten und Rückschlüsse auf Ihren neuen ML-Modellendpunkt zu ziehen.

## Lasst uns startet!
Sind Sie bereit? Lassen Sie uns überprüfen, ob Sie die folgenden Voraussetzungen erfüllt haben:
1. Haben Sie das [**Intelligente Thermostat**](../smart-thermostat.html) abgeschlossen?
2. Haben Sie mit einem Testclient (wie dem in der AWS IoT Core-Konsole) bestätigt, dass Sie Nachrichten von Ihrem intelligenten Thermostat sehen können? Sie sollten in der Lage sein, das Thema `$aws/things/<<CLIENT_ID>>/shadow/update/accepted` zu abonnieren (<<CLIENT_ID>> durch die Client-ID/Seriennummer Ihres Geräts zu ersetzen) und sehen, dass Nachrichten im Testclient ankommen.

Wenn ja, fahren wir mit dem nächsten Kapitel fort, [**Datenweiterleitung und -speicherung**](/de/smart-spaces/data-routing-and-storage.html).

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}