У Kubernetes есть [volumes](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir), например, нативный [emptyDir](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir). Часть из них stateless, то есть они живут, пока жив под. Судьба у данных, которые туда попадают, аналогичная.  
  
Для statefull-приложений используются постоянные хранилища, Persistent Volumes (PV). [Persistent Volumes (PV)](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) — это единицы хранения, которые были выделены кластеру Kubernetes его администратором. Это могут быть локальные диски, СХД, внешние дисковые полки. Они никак не зависят от жизненного цикла подов.  
  
Persistent Volume Claim (PVC) — это запрос на выделение PV определенных характеристик: типа хранилища, объема, типа доступа (чтение и/или запись). Для описания подробных характеристик доступных PV используются [Storage Classes](https://kubernetes.io/docs/concepts/storage/storage-classes/).  
  
В динамике это все выглядит следующим образом: под отправляет PVC, а PVC уже обращается к PV и передает ее поду.

![image](https://habrastorage.org/r/w1560/webt/6f/ad/kv/6fadkvflto8baynbd2bapqxc19e.png)  
_Схема выделения PV подам_