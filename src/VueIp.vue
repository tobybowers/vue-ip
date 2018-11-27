<template lang="pug">
    span(v-if="theme === 'material'").material.vue-ip(:class="{'show-port' : portCopy !== false, 'active': active, 'valid': valid}")
        .label
            slot
        .segment(v-for="(segment, index) in ipCopy")
            input(type="number", maxlength="3", step="1"
                v-model="ipCopy[index]",
                :placeholder="placeholderPos(index)",
                @paste="paste($event)",
                @keydown="ipKeydown($event, index)",
                @focus="ipFocus(index)",
                @blur="blur",
                ref="ipSegment")
        input(type="number", 
            v-show="portCopy !== false",
            v-model="portCopy",
            :placeholder="((placeholder) ? '8080' : '')",
            @paste="paste($event)",
            @focus="portFocus",
            @keydown="portKeydown",
            @blur="blur", 
            ref="portSegment").port

    form(v-else).form-inline.vue-ip
        label.sr-only
            slot
        .form-group
            span.segment.input-group-addon(v-for="(segment, index) in ipCopy")
                input(type="number", maxlength="3",
                    v-model="ipCopy[index]",
                    :placeholder="placeholderPos(index)",
                    @paste="paste($event)",
                    @keydown="ipKeydown($event, index)",
                    @focus="ipFocus(index)",
                    @blur="blur",
                    ref="ipSegment"
                ).form-control
            input(type="number", 
                v-if="portCopy !== false",
                v-model="portCopy",
                :placeholder="((placeholder) ? '8080' : '')",
                @paste="paste($event)",
                @focus="portFocus",
                @keydown="portKeydown",
                @blur="blur", 
                ref="portSegment").form-control
</template>

<style lang="stylus" scoped>
    $ip-material-valid := #8AC63F
    $ip-material-in-valid := #f25d59

    $ip-material-color := #222222
    $ip-material-color-mute := rgba(28,128,195,0.6);
    $ip-material-fontSize := inherit

    $ip-transition-speed := .15s

    .vue-ip
        position relative
        display inline-block
        text-align left

        &.material
            transition all $ip-transition-speed ease-in-out
            border-bottom 1px solid $ip-material-color-mute
            padding 6px

            .label
                display block
                transition all $ip-transition-speed ease-in-out
                text-align left
                position absolute
                width 100%
                font-size .6rem
                top -20px
                color $ip-material-color-mute

            .segment
                &:not(:last-of-type)
                    &:after
                        content '.'
                        display inline-block
                &:after
                    color $ip-material-color

            &.active
                &.valid
                    &.material
                        border-bottom-color $ip-material-valid

                        .segment
                            &:after
                                color $ip-material-valid

                        .label
                            color $ip-material-valid

                &:not(.valid)
                    &.material
                        border-bottom-color $ip-material-in-valid

                        .segment
                            &:after
                                color $ip-material-in-valid

                        .label
                            color $ip-material-in-valid

            input
                background transparent
                font-size $ip-material-fontSize
                color $ip-material-color
                width 40px

                // Hide spinner on MOZ
                -moz-appearance: textfield

                &::-webkit-outer-spin-button, &::-webkit-inner-spin-button
                    -webkit-appearance none
                // Hide spinner on chrome

                &::placeholder
                    color rgba($ip-material-color, .6)

        

        .label
            display none

        &.show-port
            .segment
                &:last-of-type
                    &:after
                        content ':'

        .segment
            display inline-block

            &.input-group-addon
                padding 0
                width unset

        input
            text-align center
            width 50px
            outline none
            border none
            // Hide spinner on MOZ
            -moz-appearance: textfield

            &::-webkit-outer-spin-button, &::-webkit-inner-spin-button
                -webkit-appearance none
            // Hide spinner on chrome

            &.port
                width 60px

</style>

<script>
    export default {
        props: {
            onChange: Function,
            ip: {
                required: true,
                type: String
            },
            port: {
                type: [String, Number, Boolean],
                default: false
            },
            placeholder: {
                type: [Boolean],
                default: false
            },
            theme: {
                type: [String, Boolean],
                default: false
            },
            target: {
                type: [String, Boolean],
                default: false
            }
        },
        data() {
            return {
                ipCopy: ['', '', '', ''],
                portCopy: null,
                valid: false,
                active: false,
                timeout: null
            }
        },
        beforeMount() {
            // Copy the values over
            this.copyValue(this.ip, this.port);

        },
        watch: {

            /**
             * Watch the IP prop for changes and update internally
             */
            ip(newIp) {
                this.copyValue(newIp, this.port);
            },

            /**
             * Watch the port for changes and update internally
             */
            port(newPort) {
                this.copyValue(this.ip, newPort);
            }

        },
        methods: {

            /**
             * Placeholder with dummy IP
             */
            placeholderPos(segment) {

                // No placeholder
                if (!this.placeholder)
                    return '';

                // Dummy IP placeholder
                switch (segment) {
                    case 0:
                        return '192';
                    case 1:
                        return '168';
                    case 2:
                        return '0';
                    case 3:
                        return '1';
                }

            },

            /**
             * On focus clear the current box
             */
            ipFocus(index) {

                this.active = true;

                // Clear it
                this.ipCopy[index] = '';

                // Update the change
                this.changed();

            },

            /**
             * Clear both inputs
             */
            clearAll() {
                this.ipCopy = ['', '', '', ''];
                this.portCopy = null;
                this.valid = false;
            },

            /**
             * Clicked off the IP or port
             */
            blur() {

                this.active = false;

            },

            /**
             * Focus on the port
             */
            portFocus() {

                this.active = true;

                // Clear it
                this.portCopy = null;

                // Update the change
                this.changed();

            },

            /**
             * Paste in an IP (with or without a port)
             */
            paste(event) {

                // Focus on first el
                this.$refs.ipSegment[0].focus();

                // Get clipboard text
                let pasteText = event.clipboardData.getData('text/plain');

                // Check if we have a port or not
                let portPos = pasteText.indexOf(':');

                // If we have ports turned off, remove the port and only update the IP value
                if (this.port === false) {

                    //console.warn('A IP address with a port has been entered but this module has no port attribute. Please enable it update the port.');

                    this.clearAll();

                    let ipAndPort = pasteText.split(":");
                    this.copyValue(ipAndPort[0], false);

                    // Blur off input
                    this.$refs.ipSegment[0].blur();

                    return;
                }

                // Based on if we have a port or not
                switch (portPos) {
                    case -1:
                        this.copyValue(pasteText, null);
                        this.changed();

                        // Blur off input
                        this.$refs.ipSegment[0].blur();

                        break;
                    default:
                        let ipAndPort = pasteText.split(":");
                        this.copyValue(ipAndPort[0], ipAndPort[1]);
                        this.changed();

                        // Blur off input
                        this.$refs.ipSegment[0].blur();

                        break
                }

            },

            /**
             * Port keydown event
             */
            portKeydown() {

                let keyCode = event.keyCode || event.which;

                // Return or left on keypad
                if (keyCode === 8 || keyCode === 37) {

                    // If there is nothing within the selected input go back to the last IP segment
                    if (this.portCopy && this.portCopy.length === 0 && this.portCopy !== undefined)
                        this.$refs.ipSegment[3].focus();

                }

                setTimeout(this.changed);

            },

            /**
             * Run on keydown
             */
            ipKeydown(event, index) {

                clearTimeout(this.timeout);

                let keyCode = event.keyCode || event.which;

                // Backspace or left on keypad
                if (keyCode === 8 || keyCode === 37) {

                    // If there is nothing within the selected input go the the one before it
                    if (this.ipCopy[index].length === 0 && this.ipCopy[index - 1] !== undefined)
                        this.$refs.ipSegment[index - 1].focus();

                }

                // else if (keyCode === 110 || keyCode === 190) {
                //     console.log('dot');
                //     // Allow tabing to next segment using Tab, Period or Enter if current segment has entry
                //     if (this.ipCopy[index].length >= 1) {
                //         this.moveToNextIpSegment(index, false);
                //         this.ipCopy[index + 1] = this.ipCopy[index + 1].substring(0, this.ipCopy[index + 1].length - 1);
                //     }
                // }

                // Semi-colon (jump to port number)
                else if (keyCode === 186)
                    this.$refs.portSegment.focus();


                this.timeout = setTimeout(() => {
                    //console.log('Timeout');
                    // If its a 0 then always move to the next segment, if not work out if we need to move first
                    if (this.ipCopy[index] === '0') {
                        this.moveToNextIpSegment(index, false);
                    } else {
                        this.moveToNextIpSegment(index);
                    }
                        

                    // Update the change
                    this.changed();

                }, 200);
            },

            /**
             * Work out if we need to move to the next IP segment or not
             */
            moveToNextIpSegment(index, ifOverThree = true) {

                /**
                 * If there is 3 characters check if there is another segment, if there is focus on it.
                 */
                if (ifOverThree) {

                    if (this.ipCopy[index].length >= 3 && this.ipCopy[index + 1] !== undefined)
                        this.$refs.ipSegment[index + 1].focus();
                    else if (this.ipCopy[index].length >= 3 && this.ipCopy[index + 1] === undefined)
                        this.$refs.portSegment.focus();

                } else if (!ifOverThree) {

                    if (this.ipCopy[index + 1] !== undefined)
                        this.$refs.ipSegment[index + 1].focus();
                    else if (this.ipCopy[index + 1] === undefined)
                        this.$refs.portSegment.focus();

                }

            },

            /**
             * Update the controller with changed IP and port addresses
             */
            changed(ip = this.ipCopy, port = this.portCopy) {
                let ipLocal = this.arrayToIp(ip);
                this.ip = ipLocal;
                this.onChange(ipLocal, port, this.validateIP(ip), this.target);
            },

            /**
             * Copy prop into local copy
             */
            copyValue(ip, port) {

                if (ip)
                    this.ipToArray(ip);

                // Update the port as long as its a number
                this.portCopy = port;

                // Update if its valid locally
                this.valid = this.validateIP(this.ipCopy);

                // Report right back with if its valid or not
                this.changed();

            },

            /**
             * Convert the IP address string to an array of values
             */
            ipToArray(ip) {

                let segments = [];
                ip.split('.').map(segment => {
                    if (isNaN(segment) || segment < 0 || segment > 255)
                        segment = 255;
                    segments.push(segment);
                });

                // If something is not valid clear it all
                if (segments.length !== 4) {
                    //console.error('Not valid, so clearing ip', segments);
                    this.clearAll();
                } else
                    this.ipCopy = segments;

            },

            /**
             * Convert the array of IP segments back to a string
             */
            arrayToIp(ipArray) {
                return ipArray.join(".");
            },

            /**
             * validates the IP address
             *
             * @returns Boolean
             */
            validateIP(ip) {
                let ipCheck = this.arrayToIp(ip);
                return (/^(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$/.test(ipCheck))
            }

        }
    }
</script>
