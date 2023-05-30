# Optics for Java

* Lens
* Optional
* Iso
* Traversal
* Fold


# Annotation Processor that automatically generates the optics for simple records

This is a POC to see what the limitations are.

It only works with Java records at the moment. A companion object is created which contains the optics. It contains
lens for each field, and a traversal for fields that are list/collection or stream.

There is an experimental feature to generate chained traversals, but unfortunately this is problematic with
incremental compilation. A new approach would be needed to make this work.

## @Optic annotation

This can be added to any record. It will
* Create a companion object with optics for each field
  * Lens for each field
  * Traversal for each field that is a list/collection or stream
* If addListTraversal is true, then a traversal is added that traverses over a list of the record type
  * This is useful when business methods have a list of the records 
* The 'traversals' field allows chains of traversals to be generated in the companion object
  * See the example below for examples.
  * Note that a traversal chain can be given a name as shown in the second array element in the example below.

## Example

```java
@Optics(debug = false, addListTraversal = true, traversals = {
        "commandeTransportList.tronconTransportList",
        "commande2ChassisT:commandeTransportList.tronconTransportList.chassisTronconList"
})
public record Commande(List<CommandeTransport> commandeTransportList) {
}
```
This generated the companion object:
```java
public interface CommandeOptics extends IGeneratedOptics<Commande> {
 ILens<Commande,List<CommandeTransport>> commandeTransportListL =
    ILens.of(Commande::commandeTransportList,(main,value)->new Commande(value));

  ITraversal<Commande, CommandeTransport> commandeTransportListT =
    ITraversal.fromListLens(commandeTransportListL);

  ITraversal<List<Commande>,Commande> listT=ITraversal.listTraversal();
  ITraversal<Commande,TronconTransport> commandeTransportList_tronconTransportListT = CommandeOptics.commandeTransportListT
    .andThen(CommandeTransportOptics.tronconTransportListT) ;

  ITraversal<Commande,ChassisTroncon> commande2ChassisT = CommandeOptics.commandeTransportListT
    .andThen(CommandeTransportOptics.tronconTransportListT)
    .andThen(TronconTransportOptics.chassisTronconListT) ;
}
``` 


    
