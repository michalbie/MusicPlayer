<template>
    <div class="song-bar-wrapper">
        <div v-if="!isPlaylist" style="display:flex; align-items: center; margin-right: 0.2em;">
            <img class="add-btn" src="../../assets/add.png" @click="togglePlaylistsMenu" />
        </div>
        <div v-else style="display:flex; align-items: center; margin-right: 0.2em;">
            <img class="add-btn" src="../../assets/delete.png" @click="removeSongFromPlaylist" />
        </div>
        <div class="first-half-wrapper">
            <img v-if="isCurrentlyPlaying" class="play-btn" src="../../assets/pause.png" @click="pauseSong" />
            <img v-else class="play-btn" src="../../assets/play.png" @click="playSong" />
            <p class="song-title">{{ this.songTitle }}</p>
        </div>

        <div class="song-data-container">
            <p class="album-name">{{ this.albumName }}</p>
            <p class="song-size">{{ this.size }} MB</p>
        </div>

        <playlists-choice-popup
            :song-title="this.songTitle"
            :album-name="this.albumName"
            :size="this.size"
            class="choice-popup"
        ></playlists-choice-popup>
    </div>
</template>

<script>
import PlaylistsChoicePopup from "../PlaylistsChoicePopup.vue";

export default {
    name: "SongBar",
    props: ["songTitle", "albumName", "size"],
    computed: {
        getSongTitle() {
            return this.songTitle;
        },
        getAlbumName() {
            return this.$store.state.currentAlbum;
        },
        isCurrentlyPlaying() {
            if (this.$store.state.currentSongPlaying == this.songTitle && this.$store.state.isPlaying == true) {
                return true;
            }
            return false;
        },
        isPlaylist() {
            let isPlaylist = false;
            this.$store.state.playlists.forEach(el => {
                if (el.playlistName == this.albumName.replace(/\s/g, "")) {
                    //innerHTML contains spaces - I delete them
                    isPlaylist = true;
                }
            });
            return isPlaylist;
        }
    },
    methods: {
        playSong: function() {
            this.$store.state.playingAlbumSongs = this.$store.state.songs;
            let albumName = this.albumName;
            if (this.albumName[0] == " " && this.albumName[this.albumName.length - 1] == " ") {
                albumName = this.albumName.slice(1, this.albumName.length - 1);
            }

            //if it is a playlist, assign album name individually for every song (instead of one for all songs)
            this.$store.state.playlists.forEach(el => {
                if (el.playlistName == albumName) {
                    this.$store.state.isPlaylistPlaying = true;
                    this.$store.state.currentPlaylistPlaying = el.playlistName;
                    for (let i = 0; i < el["files"].length; i++) {
                        if (el["files"][i].file == this.songTitle) {
                            albumName = el["files"][i].albumName;
                        }
                    }
                }
            });

            //if no playlist is playing - change isPlaylistPlaying to false
            if (albumName == this.albumName) {
                this.$store.state.isPlaylistPlaying = false;
                this.$store.state.currentPlaylistPlaying = null;
            }

            this.$store.dispatch({
                type: "playSong",
                songName: this.songTitle,
                albumName: albumName
            });
        },

        pauseSong: function(event) {
            this.$store.dispatch("pauseSong");
            event.target.src = "../../assets/play.png";
        },

        togglePlaylistsMenu: function() {
            //get particular popup
            let popup = this.$el.getElementsByClassName("choice-popup")[0];

            //remove all previous popups
            let allOtherPopups = document.getElementsByClassName("choice-popup");
            allOtherPopups.forEach(element => {
                if (element != popup) {
                    element.style.display = "none";
                }
            });

            //then create new one
            if (popup.style.display == "flex") {
                popup.style.display = "none";
            } else {
                popup.style.display = "flex";
            }
        },

        removeSongFromPlaylist: function() {
            this.$store.dispatch({
                type: "removeSongFromPlaylist",
                songName: this.songTitle,
                playlistName: this.albumName.replace(/\s/g, "")
            });
        }
    },
    components: {
        "playlists-choice-popup": PlaylistsChoicePopup
    },
    mounted: function() {
        let popup = this.$el.getElementsByClassName("choice-popup")[0];
        popup.addEventListener("mouseleave", () => {
            popup.style.display = "none";
        });
    }
};
</script>

<style lang="scss" scoped>
@import "../styles/mixins.scss";

.song-bar-wrapper {
    position: relative;
    display: flex;
    justify-content: space-between;
    padding: 0.4em 0.6em 0.3em 0.4em;
    margin-top: 0.1em;
    background-color: rgba(10, 10, 10, 0.1);
    border-radius: 0.5em;
    margin-right: 0.5em;

    @include w700 {
        font-size: 0.8em;
    }

    @include w767P {
        font-size: 0.8em;
    }

    .first-half-wrapper {
        display: flex;
        flex: 1;
        align-items: center;

        @include w767P {
            overflow: hidden;
        }

        .play-btn {
            width: 1em;
            height: 1em;
            transition: all 0.5s ease;
        }

        .play-btn:hover {
            cursor: pointer;
            transform: scale(1.1);
        }

        .song-title {
            margin-left: 0.5em;
            font-size: 0.8em;

            @include w767P {
                overflow: hidden;
                text-overflow: ellipsis;
                white-space: nowrap;
            }
        }
    }

    .add-btn {
        width: 1.1em;
        height: 1.1em;
        transition: all 0.5s ease;
    }

    .add-btn:hover {
        cursor: pointer;
        transform: scale(1.1);
    }

    .song-data-container {
        display: flex;
        justify-content: space-between;
        align-items: center;
        margin-left: 4em;
        width: 30%;
        font-size: 0.8em;

        @include w1500 {
            justify-content: flex-end;
        }

        @include w1024P {
            justify-content: flex-end;
        }

        @include w767P {
            display: none;
        }

        .album-name {
            color: #2b2836;

            @include w767P {
                overflow: hidden;
                text-overflow: ellipsis;
                white-space: nowrap;
            }
        }

        .song-size {
            @include w1500 {
                display: none;
            }

            @include w1024P {
                display: none;
            }
        }
    }
}
</style>
