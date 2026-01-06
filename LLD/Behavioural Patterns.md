# Iterator
- separating the logic of how you move through a collection from the collection itself.
## Example - Iterating through a Playlist
```Java
class Playlist {
    private List<String> songs = new ArrayList<>();
    public void addSong(String song) {
        songs.add(song);
    }
    public List<String> getSongs() {
        return songs;
    }
}
class MusicPlayer {
    public void playAll(Playlist playlist) {
        for (String song : playlist.getSongs()) {
            System.out.println("Playing: " + song);
        }
    }
}
// This should not be possible, but it is
playlist.getSongs().add("Unauthorized Song");
playlist.getSongs().remove(0);
```
## Problems
- **Breaks Encapsulation**
- **Tightly couples client to implementation**
- **Limited Traversal Options**
- **Testing becomes difficult**
## Implementing Iterator
```Java
interface Iterator<T>{
	boolean hanNext();
	T next();
}
interface IterableCollection<T> {
    Iterator<T> createIterator();
}
class Playlist implements IterableCollection<String> {
    private final List<String> songs = new ArrayList<>();
    public void addSong(String song) {
        songs.add(song);
    }
    public String getSongAt(int index) {
        return songs.get(index);
    }
    public int getSize() {
        return songs.size();
    }
    @Override
    public Iterator<String> createIterator() {
        return new PlaylistIterator(this);
    }
}
class PlaylistIterator implements Iterator<String> {
    private final Playlist playlist;
    private int index = 0;
    public PlaylistIterator(Playlist playlist) {
        this.playlist = playlist;
    }
    @Override
    public boolean hasNext() {
        return index < playlist.getSize();
    }
    @Override
    public String next() {
        return playlist.getSongAt(index++);
    }
}
public class MusicPlayer {
    public static void main(String[] args) {
        Playlist playlist = new Playlist();
        playlist.addSong("Shape of You");
        playlist.addSong("Bohemian Rhapsody");
        playlist.addSong("Blinding Lights");
        Iterator<String> iterator = playlist.createIterator();
        System.out.println("Now Playing:");
        while (iterator.hasNext()) {
            System.out.println(" 🎵 " + iterator.next());
        }
    }
}
```
## Class Diagram
![[Screenshot 2026-01-05 at 11.29.44 PM.png|700]]
## Benefits
- **Encapsulation is preserved**
- **Implementation independence** - The client code works with the Iterator interface.
- **Single Responsibility Principle** - Playlist class focuses on managing songs. PlaylistIterator class focuses on traversal logic.
- **Multiple simultaneous traversals** - create iterator returns a new iterator everytime.
- **Foundation for extensions** - We can now easily add new types of iterators
# Observer
- If the state of one object changes, all the other objects are notified and updated.
- Defines a one to many relationship.
- 