+++
title = "Einführung"
weight = 10
pre = "<b>a. </b>"
+++

## Ihre Aufgabe

In diesem Szenario nehmen Sie die Rolle eines Full-Stack-Entwicklers auf, der damit beauftragt ist, die Thermostatfunktionen eines Besprechungsraums zu automatisieren, um Energie zu sparen. Jetzt untersuchen Sie, wie Sie den Mechanismus zur Bestimmung, ob der Raum belegt ist, über den einfachen Geräuschschwellenwert aus dem vorherigen Modul hinaus verbessern können. Ein einfaches maschinelles Lernmodell könnte Ihnen dabei helfen, eine intelligentere Klassifizierung als einen statischen numerischen Schwellenwert zu identifizieren und die Daten der vergangenen Raumbelegung zu berücksichtigen, um Jitter oder False-Positives zu reduzieren.

Das Trainieren eines Machine-Learning-Modells ist heutzutage mit modernen Data-Science Werkzeugen relativ einfach. Das Trainieren eines _guten_ Modells ist jedoch schwierig und erfordert Fachwissen, ein umfassendes Verständnis der Eingaben und Zyklen zur Optimierung. Heute lernen Sie die Amazon SageMaker Werkzeuge kennen, um die IoT-Daten zum Trainieren eines Modells verwenden zu könne. Aber wir können nicht erwarten, dass beim ersten Versuch ein _gutes_ Modell erzeugt wird.

Bitte beachten Sie, dass die Fertigstellung dieses Moduls etwa zwei Tage in Anspruch nimmt. Obwohl Sie nur eine Stunde praktische Arbeit leisten müssen, gibt es zwei Schritte, bei denen Sie die Hände frei haben werden. Der erste Schritt ist der Einsatz Ihres intelligenten Thermostatgeräts in dem Raum, den Sie untersuchen möchten, und das Sammeln von Telemetriedaten zum Speichern für das ML-Training. Sie sollten Ihr Gerät mindestens ein paar Stunden lang einsetzen und Daten sammeln (vorzugsweise 24 Stunden), aber je mehr Daten Sie sammeln, desto genauer kann das ML-Modell sein. Im zweiten Schritt durchläuft das ML-Modell einen automatischen Trainingsprozess, sobald Sie genügend Daten gesammelt haben. Dieser Trainingsprozess kann mehrere Stunden dauern und der wir empfehlen, ihn am Morgen zu starten und am Nachmittag zurückzukehren oder ihn über Nacht laufen zu lassen.

## Problemlösung

Die im letzten Modul erstellte Lösung verwendete einen statischen numerischen Schwellenwert für den eingehenden Schallpegel, um einen neuen Schlüsselwert namens *roomOccupancy* zu erzwingen. Um ein einfaches maschinelles Lernmodell zu trainieren, das eine ähnliche Funktion ausführt, werden Sie vorhandene Daten als Basis für das Training verwenden. Das bedeutet, dass Sie die vorhandene Lösung mehrere Stunden lang in einem Raum laufen lassen müssen, der abwechselnd den Zustand hat, dass der Raum aktiv besetzt ist oder nicht. Sie werden einen aggregierten Datensatz dieser Gerätetelemetrie verwenden, um ein automatisches ML-Trainingsexperiment durchzuführen, das dann verwendet wird, um neue Gerätemeldungen mit einem neuen *roomOccupancy* Wert zu klassifizieren, der von Ihrem trainierten ML-Modell abgeleitet wird. Auch hier ist Ihr erstes trainiertes Modell vielleicht nicht ganz so genau, aber der Zweck dieser Lösung ist es, Ihnen praktische Erfahrungen mit dem Prozess des Speicherns von IoT-Daten und dem Üben des Arbeitsablaufs des Trainings und der Nutzung eines neuen ML-Modells zu vermitteln.

## Lösungsarchitektur
![Smart Spaces architecture](introduction/smartspace-overview.png)

Der Arbeitsablauf, den Sie in diesem Modul aufbauen werden, hat die folgenden Hauptkomponenten:

1. Eine neue IoT-Regel leitet Geräte-Schatten-Updates an einen Dienst namens AWS IoT Analytics weiter. IoT Analytics ist ein Service zum Speichern von IoT-Rohdaten in großen Mengen, zum Durchführen von Transformationen und Bereinigungsvorgängen, um Rohdaten in verarbeitete Daten umzuwandeln, und zum Bereitstellen einer Abfrage-Engine für das Slicen von verarbeiteten Daten für analytische Workflows oder ML-Training.
2. Ihr IoT-Analytics-Projekt besteht aus vier miteinander verknüpften Ressourcen: einem Kanal zum Speichern von IoT-Rohdaten aus dem Geräteschatten, einer Pipeline für Datentransformationen/Filterung/Anreicherung, einem Datenspeicher für verarbeitete Daten und einem Datensatz, der gespeicherte Abfragen ausführt und die Ergebnisse zur Verarbeitung senden kann.
3. Amazon SageMaker Studio ist eine integrierte Umgebung für maschinelles Lernen, in der Sie Ihre Modelle erstellen, trainieren, bereitstellen und analysieren können - alles in derselben Anwendung.
4. Ein Amazon SageMaker-Endpunkt, der Ihr trainiertes Modell als abrufbare API bereitstellt.
5. Eine AWS Lambda-Funktion, die einen einfachen Code ausführt, um Ihre von Thermostat veröffentlichten Nachrichten zu verarbeiten und Rückschlüsse auf Ihren neuen ML-Modell-Endpunkt zu ziehen.

## Los geht's!

Sind Sie bereit, mit dem Bau der Lösung zu beginnen? Lassen Sie uns überprüfen, ob Sie die folgenden Voraussetzungen erfüllt haben:

1. Haben Sie das vorherige Modul in dieser Reihe von Tutorials mit dem Titel **Intelligentes Thermostat** abgeschlossen?
2. Wissen Sie, an welchem seriellen Port Ihr Core2 for AWS IoT EduKit-Gerät angeschlossen ist? Dies wird auch im **Blinky Hello World** Tutorial behandelt. Sie werden es in mehrere der Schritte dieses Tutorials für Platzhalterwerte wie `<<DEVICE_PORT>>` einfügen.
3. Haben Sie bestätigt, dass Sie Nachrichten sehen können, die von Ihrem intelligenten Thermostat ankommen, indem Sie einen Test-Client wie den in der AWS IoT Core-Konsole verwenden? Sie sollten in der Lage sein, das Topic `$aws/things/<<CLIENT_ID>>/shadow/update/accepted` zu abonnieren (wobei Sie <<CLIENT_ID>> durch die Client-ID/Seriennummer Ihres Geräts ersetzen) und zu sehen, dass Nachrichten im Test-Client ankommen.

Wenn ja, lassen Sie uns mit dem nächsten Kapitel, [Datenrouting und -speicherung,](/de/smart-spaces/data-routing-and-storage.html) fortfahren.

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://community.m5stack.com/category/41/core2-for-aws" icon="far fa-question-circle" %}}Community support{{% /button %}}