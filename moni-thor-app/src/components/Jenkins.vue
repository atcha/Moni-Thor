<template>
    <div class="jenkins ">
        <md-app-toolbar class=" md-toolbar" style="height: 50px;">
            <span>
                <h2>Jenkins</h2>
            </span>
        </md-app-toolbar>
        <div>
            <md-progress-bar class="md-primary" md-mode="indeterminate" v-if="loading"></md-progress-bar>
            <md-tabs @md-changed="initView">
                <md-tab v-for="build in JenkinsStore.builds"
                        :id="build.name"
                        :md-label="build.name"
                        :md-icon="getIcon(build)"
                >
                    <md-list v-if="build">
                        <md-list-item>
                            <div class="md-layout" style="font-weight: bold;width: 100%;">
                                <div class="md-layout-item md-size-40">
                                    Job
                                </div>
                                <div class="md-layout-item md-size-20">
                                    Time of build
                                </div>
                                <div class="md-layout-item md-size-20">
                                    Version
                                </div>
                                <div class="md-layout-item md-size-2">
                                    Build by Me
                                </div>
                                <div class="md-layout-item md-size-18">
                                    Last builder
                                </div>
                            </div>
                        </md-list-item>
                        <md-list-item v-for="job in build.jobs">
                            <div class="md-layout" style="width: 100%">
                                <div class="md-layout-item md-size-40"
                                     v-if="job.infos && job.infos.lastBuild && job.infos.lastBuild.url">
                                    <span class="state" v-bind:class="job.color">
                                            <i class="material-icons" v-if="job.color === 'blue'">check</i>
                                            <i class="material-icons"
                                               v-if="job.color === 'yellow' || job.color === 'red'">add_alert</i>
                                            <i class="material-icons" v-if="job.color.includes('anime')">refresh</i>
                                            <a class="terminal" title="console" target="_blank"
                                               :href="getConsoletUrl(job)">
                                                <i class="material-icons">video_label</i>
                                            </a>
                                            <i class="material-icons" v-if="!job.color.includes('anime')" style="color: #88c342;cursor:pointer;" title="Launch build"
                                            v-on:click="launchBuild(job)">play_circle_filled</i>
                                    </span>
                                    <span>
                                        <a target="_blank" :href="job.url">{{job.name}}</a>
                                    </span>
                                </div>
                                <div class="md-layout-item md-size-20">
                                    <span v-if="job.infos.lastBuild && job.infos.lastBuild.timestamp">{{formatDate(job.infos.lastBuild.timestamp)}}</span>
                                </div>
                                <div class="md-layout-item md-size-20">
                                    <span v-if="job.infos">
                                        <label class="version"
                                               v-if="job.infos.maven && job.infos.maven.moduleRecords && job.infos.maven.moduleRecords[0]">
                                          {{job.infos.maven.moduleRecords[0].pomArtifact.version}}
                                        </label>
                                    </span>
                                </div>
                                <div class="md-layout-item md-size-2">
                                    <div v-if="job.infos.lastBuild && job.infos.lastBuild.culprits && job.infos.lastBuild.culprits[0]">
                                        <i v-if="isConnectedUser(job.infos.lastBuild.culprits[0].fullName)"
                                           class="material-icons">star</i>
                                    </div>
                                </div>
                                <div class="md-layout-item md-size-18">
                                    <span v-if="job.infos.lastBuild && job.infos.lastBuild.culprits && job.infos.lastBuild.culprits[0]">
                                        <label class="culprit">
                                            {{job.infos.lastBuild.culprits[0].fullName}}
                                        </label>
                                    </span>
                                </div>
                            </div>
                        </md-list-item>
                    </md-list>
                </md-tab>
            </md-tabs>
        </div>
    </div>
</template>

<script>
    import Vue from 'vue'
    import * as VueMaterial from "vue-material";
    import JenkinsStore from "../store/JenkinsStore";

    export default {
        name: 'jenkins',
        data() {
            return {
//                url : '/jenkins',
                user: '',
                views: [],
                builds: [],
                tab: {
                    data: ''
                },
                loading: true,
                JenkinsStore : JenkinsStore
            }
        },
        computed: {},
        methods: {
            getConsoletUrl(job) {
                return job.infos.lastBuild.url + 'console';
            },
            formatDate(timestamp) {
                let date = new Date(timestamp);
                return this.twoDigits(date.getDate()) + '/' +
                    this.twoDigits((date.getMonth() + 1)) + '/' +
                    date.getFullYear() + ' ' +
                    this.twoDigits(date.getHours()) + ':' +
                    this.twoDigits(date.getMinutes());
            },
            twoDigits(time) {
                return time < 10 ? '0' + time : time;
            },
            getIcon(build) {
                if (build.jobs.filter((job) => job.color.startsWith('red') || job.color.startsWith('yellow')).length > 0) {
                    return 'add_alert';
                }
                if(build.jobs.filter((job) => job.color.includes('anime')).length > 0){
                    return 'refresh';
                }
                return 'done';
            },
            initView(id) {

            },
            isConnectedUser(username) {
                if (username && JenkinsStore.ldapUser) {
                    return username.includes(JenkinsStore.ldapUser);
                }
            },
            launchBuild(job){
                this.JenkinsStore.launchBuild(job);
            }
        },
        mounted() {
            this.loading = true;
            JenkinsStore.initUser().then(() => {
                JenkinsStore.initJenkins().then(() => this.loading = false).catch(() => this.loading = false);
            });
        }
    }
</script>
<style>
    h1, h2 {
        font-weight: normal;
    }

    nav.div.md-tabs-navigation {
        padding: 0px;
    }

    .jenkins span.state.blue {
        color: cornflowerblue;
    }

    .jenkins span.state.yellow {
        color: #ffcc3c;
    }

    .add_alert {
        color: darkred;
    }

    .jenkins span.state.red {
        color: darkred;
    }

    .jenkins span.state.red_anime > i,
    .jenkins span.state.yellow_anime > i,
    .jenkins span.state.blue_anime > i{
        /*background-image: -webkit-radial-gradient(20px 20px, circle cover, white, cornflowerblue);*/
        animation-name: spin;
        animation-duration: 3s; /* 3 seconds */
        animation-iteration-count: infinite;
        animation-timing-function: linear;
    }

    @keyframes spin {
        from {
            transform: rotate(0deg);
        }
        to {
            transform: rotate(360deg);
        }
    }

    .md-list-item .md-list-item-container:hover {
        text-decoration: none;
        background-color: whitesmoke;
    }

    .md-list-item:last-child .md-list-item-container {
        border-bottom: none;
    }

    .md-list-item .md-list-item-container {
        border-bottom: 1px solid lightgrey;
    }

    .connected {
        font-weight: bold;
    }

    .jenkins .md-tabs .md-tabs-container {
        transition: none;
    }

    .jenkins a.terminal {
        color: #000;
    }

    .jenkins .md-layout {
        align-items: center;
    }

    .jenkins .md-tabs .md-tabs-navigation .md-button:last-of-type {
        position: absolute;
        right: 0px;
        height: 100%;
    }

    .jenkins .md-tabs > .md-tabs-navigation .md-tab-header.md-active, .md-tabs > .md-tabs-navigation .md-tab-header:focus {
        background-color: #ff5722;
    }

    .md-app .jenkins .md-content.md-theme-default {
        height: auto !important;
    }

</style>
