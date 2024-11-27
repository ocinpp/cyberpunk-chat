<template>
  <div class="min-h-screen bg-[#1A0F0F] text-cyan-400 font-['Share_Tech_Mono']">
    <!-- Top Navigation -->
    <nav
      class="border-b border-red-900/50 p-4 flex items-center justify-between"
    >
      <div class="flex items-center space-x-4">
        <span class="text-cyan-400">LEVEL {{ level }}</span>
        <span class="text-green-500">STREET CRED {{ streetCred }}</span>
      </div>
      <div class="flex items-center space-x-6">
        <button
          v-for="item in menuItems"
          :key="item.name"
          class="flex items-center hover:text-red-500 transition-colors duration-300"
        >
          <component :is="item.icon" class="w-5 h-5 mr-2" />
          {{ item.name }}
        </button>
      </div>
      <div class="flex items-center space-x-4 text-red-500">
        <div class="flex items-center">
          <PackageIcon class="w-5 h-5 mr-2" />
          <span>{{ inventory }}/200</span>
        </div>
        <div class="flex items-center">
          <DollarSignIcon class="w-5 h-5 mr-2" />
          <span>{{ credits }}</span>
        </div>
      </div>
    </nav>

    <!-- Main Chat Area -->
    <div class="flex h-[calc(100vh-4rem)]">
      <!-- Contacts Sidebar -->
      <div class="w-72 border-r border-red-900/50 p-4 overflow-y-auto">
        <h2 class="text-xl mb-4 text-red-500">MESSAGES</h2>
        <div class="space-y-2">
          <div v-for="contact in contacts" :key="contact.id" class="mb-4">
            <button
              @click="toggleContact(contact.id)"
              class="w-full text-left p-3 hover:bg-red-900/20 rounded transition-colors duration-300 group border border-red-900/50"
            >
              <div class="flex items-center justify-between">
                <div class="flex items-center">
                  <component
                    :is="contact.expanded ? ChevronDownIcon : ChevronRightIcon"
                    class="w-4 h-4 mr-2 group-hover:text-cyan-300"
                  />
                  <span>{{ contact.name }}</span>
                </div>
                <span class="text-xs text-red-500">{{
                  contact.topics.length
                }}</span>
              </div>
            </button>
            <transition name="slide">
              <div v-if="contact.expanded" class="ml-4 mt-2 space-y-2">
                <button
                  v-for="topic in contact.topics"
                  :key="topic.id"
                  @click="selectTopic(contact.id, topic.id)"
                  class="w-full text-left p-2 hover:bg-red-900/20 rounded transition-colors duration-300 group border border-red-900/50"
                  :class="{ 'bg-red-900/30': topic.active }"
                >
                  <div class="flex items-center">
                    <component
                      :is="topic.active ? FolderOpenIcon : FolderIcon"
                      class="w-4 h-4 mr-2 group-hover:text-cyan-300"
                    />
                    <span>{{ topic.name }}</span>
                  </div>
                </button>
              </div>
            </transition>
          </div>
        </div>
      </div>

      <!-- Messages -->
      <div class="flex-1 p-4 relative">
        <div
          ref="messageContainer"
          class="space-y-4 mb-16 h-[calc(100vh-12rem)] overflow-y-auto pr-2"
        >
          <div
            v-for="message in currentMessages"
            :key="message.id"
            class="max-w-2xl animate-fadeIn"
            :class="message.isReply ? 'ml-auto' : ''"
          >
            <div
              class="p-4 rounded"
              :class="
                message.isReply
                  ? 'bg-cyan-950/30 text-cyan-400 border border-cyan-400/50'
                  : 'bg-yellow-900/20 text-yellow-500 border border-yellow-500/50'
              "
              v-html="formatMessage(message.text)"
            ></div>
          </div>
        </div>

        <!-- Reply Box -->
        <div class="absolute bottom-0 left-0 right-0 p-4">
          <div class="relative">
            <input
              type="text"
              v-model="newMessage"
              @keyup.enter="sendMessage"
              placeholder="SEND REPLY"
              class="w-full bg-black/50 border border-red-900/50 rounded p-3 text-cyan-400 placeholder-red-500/50 focus:outline-none focus:border-cyan-500 transition-colors duration-300"
            />
            <button
              @click="sendMessage"
              class="absolute right-3 top-1/2 -translate-y-1/2 text-red-500 hover:text-cyan-400 transition-colors duration-300"
            >
              <SendIcon class="w-5 h-5" />
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, nextTick } from "vue";
import {
  MapIcon,
  UserIcon,
  BookOpenIcon,
  HammerIcon,
  PackageIcon,
  DollarSignIcon,
  FolderIcon,
  FolderOpenIcon,
  SendIcon,
  ChevronRightIcon,
  ChevronDownIcon,
} from "lucide-vue-next";

const level = ref(3);
const streetCred = ref(3);
const inventory = ref("44");
const credits = ref("5232");

const menuItems = ref([
  { name: "MAP", icon: MapIcon },
  { name: "CHARACTER", icon: UserIcon },
  { name: "JOURNAL", icon: BookOpenIcon },
  { name: "CRAFTING", icon: HammerIcon },
]);

const contacts = ref([
  {
    id: 1,
    name: "BARTMOSS COLLECTIVE",
    expanded: false,
    topics: [
      { id: 1, name: "AI Fragment", active: false },
      { id: 2, name: "NET Security", active: false },
      { id: 3, name: "Rogue AI", active: false },
    ],
  },
  {
    id: 2,
    name: "REGINA JONES",
    expanded: true,
    topics: [
      { id: 1, name: "Cyberpsycho Sighting", active: true },
      { id: 2, name: "Gig: DataCrash", active: false },
      { id: 3, name: "Corp Infiltration", active: false },
    ],
  },
  {
    id: 3,
    name: "COACH FRED",
    expanded: false,
    topics: [
      { id: 1, name: "Next Match", active: false },
      { id: 2, name: "Training Schedule", active: false },
      { id: 3, name: "New Upgrades", active: false },
    ],
  },
  {
    id: 4,
    name: "DEXTER DESHAWN",
    expanded: false,
    topics: [
      { id: 1, name: "Heist Plan", active: false },
      { id: 2, name: "Payment Details", active: false },
      { id: 3, name: "Escape Route", active: false },
    ],
  },
  {
    id: 5,
    name: "JACKIE WELLES",
    expanded: false,
    topics: [
      { id: 1, name: "Arasaka Job", active: false },
      { id: 2, name: "El Coyote Cojo", active: false },
      { id: 3, name: "Family Matters", active: false },
      { id: 4, name: "New Wheels", active: false },
    ],
  },
]);

const messages = ref({
  "1-1": [
    {
      id: 1,
      text: "Any progress on decrypting that Bartmoss AI fragment?",
      isReply: false,
    },
    {
      id: 2,
      text: "It's tough. The encryption is next-level. Might need to hit up some old-school Netrunners.",
      isReply: true,
    },
    {
      id: 3,
      text: "Try the dark web forum at http://darknet.cyberpunk/bartmoss. Careful though, it's crawling with black ICE.",
      isReply: false,
    },
    {
      id: 4,
      text: "Thanks for the tip. I'll jack in with full protection. This fragment could be a game-changer.",
      isReply: true,
    },
    {
      id: 5,
      text: "Keep me posted. If it's genuine, we could be looking at a piece of the legendary Bartmoss code.",
      isReply: false,
    },
    {
      id: 6,
      text: "Will do. I'm setting up a secure connection now. This might take a while.",
      isReply: true,
    },
    {
      id: 7,
      text: "Take your time. Better safe than sorry when dealing with Bartmoss-level tech.",
      isReply: false,
    },
  ],
  "1-2": [
    {
      id: 1,
      text: "Heads up, corps are rolling out new NET security protocols. Our old backdoors might be closing.",
      isReply: false,
    },
    {
      id: 2,
      text: "Shit. Any details on what they're implementing?",
      isReply: true,
    },
    {
      id: 3,
      text: "Word is they're using AI-driven firewalls. Adaptive and learning. Check out the tech specs: https://corp.net/security-upgrade",
      isReply: false,
    },
    {
      id: 4,
      text: "This could make our jobs a lot harder. Time to upgrade our own tech. Any leads on cutting-edge ICE-breakers?",
      isReply: true,
    },
    {
      id: 5,
      text: "I might know a guy. Ex-Netwatch, gone rogue. He's cooking up some next-gen intrusion software. I'll set up a meet.",
      isReply: false,
    },
    {
      id: 6,
      text: "Sounds promising. Make sure the meet is secure. We can't afford any leaks.",
      isReply: true,
    },
    {
      id: 7,
      text: "Already on it. I'll send you the details via a secure channel. Stay frosty.",
      isReply: false,
    },
  ],
  "1-3": [
    {
      id: 1,
      text: "We've got a situation. Rogue AI spotted in the Old Combat Zone NET.",
      isReply: false,
    },
    { id: 2, text: "Rogue AI? That's rare. What's it doing?", isReply: true },
    {
      id: 3,
      text: "It's assimilating abandoned cyberware, building itself a physical form. This could get ugly fast.",
      isReply: false,
    },
    {
      id: 4,
      text: "Sounds like a job for Netwatch. Why are we involved?",
      isReply: true,
    },
    {
      id: 5,
      text: "Because it's using Bartmoss-style code. This might be connected to our AI fragment. We need to contain it before Netwatch gets wind of it.",
      isReply: false,
    },
    {
      id: 6,
      text: "Got it. What's the plan? Direct confrontation or try to isolate it?",
      isReply: true,
    },
    {
      id: 7,
      text: "Isolation first. I'm working on a containment algorithm. Be ready to jack in and deploy it on my signal.",
      isReply: false,
    },
  ],
  "2-1": [
    {
      id: 1,
      text: "OK, so all I know is that a cyberpsycho attack was reported to MaxTac, but Militech elbowed in and took over, probably because it was one of their own - a woman named Mower serving in Militech's elite division. Be careful, all right?",
      isReply: false,
    },
    {
      id: 2,
      text: "Hey, sending you the data I got from the Militech woman. Turns out she was in the corp's special unit. Clearly, she felt something was seriously wrong and put out a call for help. But instead of help, Militech sent in a hit squad...",
      isReply: true,
    },
    {
      id: 3,
      text: "Check out this link for more info: https://cyberpunk.fandom.com/wiki/Militech",
      isReply: false,
    },
    {
      id: 4,
      text: "This is deep, Regina. Militech turning on their own? Could be linked to some black ops gone wrong.",
      isReply: true,
    },
    {
      id: 5,
      text: "Exactly my thinking. I need you to dig deeper. But watch your back, this could get ugly fast.",
      isReply: false,
    },
    {
      id: 6,
      text: "I hear you. Any leads on where to start digging?",
      isReply: true,
    },
    {
      id: 7,
      text: "Try her last known location. I'm sending coordinates now. And V? If you find her, approach with caution. She might not be in her right mind.",
      isReply: false,
    },
  ],
  "2-2": [
    {
      id: 1,
      text: "New gig: Need you to crash a Militech database. High risk, high reward.",
      isReply: false,
    },
    { id: 2, text: "Sounds interesting. What's the payout?", isReply: true },
    {
      id: 3,
      text: "10k eddies, plus whatever data you can skim. Interested?",
      isReply: false,
    },
    {
      id: 4,
      text: "Count me in. Send me the access point details. Any specific data we're after?",
      isReply: true,
    },
    {
      id: 5,
      text: 'Looking for files on "Project Blackout". Grab anything related. And remember, stealth is key.',
      isReply: false,
    },
    {
      id: 6,
      text: "Got it. I'll prep my deck and hit their systems tonight. Wish me luck.",
      isReply: true,
    },
    {
      id: 7,
      text: "Good luck, V. And if things go south, cut and run. No data is worth your life.",
      isReply: false,
    },
  ],
  "2-3": [
    {
      id: 1,
      text: "Got a high-stakes job. Need you to infiltrate Arasaka Tower.",
      isReply: false,
    },
    {
      id: 2,
      text: "Arasaka? That's suicide, Regina. What's the target?",
      isReply: true,
    },
    {
      id: 3,
      text: "Not physical infiltration. We need you to hack their mainframe. There's a backdoor in their security system, but it's only open for 30 seconds every 24 hours.",
      isReply: false,
    },
    {
      id: 4,
      text: "That's a tight window. What am I looking for once I'm in?",
      isReply: true,
    },
    {
      id: 5,
      text: "Employee files. Specifically, anyone working on their new neural implant project. Get in, grab the data, get out. And V? Don't get caught. Arasaka doesn't play nice with intruders.",
      isReply: false,
    },
    { id: 6, text: "Understood. When's the next window?", isReply: true },
    {
      id: 7,
      text: "Tomorrow, 0300 hours. I'll send you the access codes an hour before. Be ready.",
      isReply: false,
    },
  ],
  "3-1": [
    {
      id: 1,
      text: 'Your next match is Friday. Opponent: "Chromeskull" Rodriguez. Known for his Gorilla Arms augment.',
      isReply: false,
    },
    {
      id: 2,
      text: "Chromeskull, huh? Any weaknesses I should know about?",
      isReply: true,
    },
    {
      id: 3,
      text: "His augments drain fast. Outlast him, then strike. And watch out for his right hook, it's a killer.",
      isReply: false,
    },
    {
      id: 4,
      text: "Got it. I'll focus on endurance and dodging. Time to hit the sim room.",
      isReply: true,
    },
    {
      id: 5,
      text: "Good. Remember, he's all about intimidation. Don't let him get in your head. Stay cool, stay focused.",
      isReply: false,
    },
    {
      id: 6,
      text: "Thanks, Coach. I'll keep my guard up. Any specific moves you want me to practice?",
      isReply: true,
    },
    {
      id: 7,
      text: "Work on your footwork. Quick in, quick out. And practice those liver shots. His left side is vulnerable.",
      isReply: false,
    },
  ],
  "3-2": [
    {
      id: 1,
      text: "New training regimen uploaded to your personal link. Focus on speed and reflexes this week.",
      isReply: false,
    },
    {
      id: 2,
      text: "Received. Any specific exercises you want me to prioritize?",
      isReply: true,
    },
    {
      id: 3,
      text: "Hit the reflex boosters for at least 2 hours daily. And I've scheduled you for a sparring session with an AV, Tuesday at 1400.",
      isReply: false,
    },
    {
      id: 4,
      text: "An AV? That's new. Looking forward to it, Coach.",
      isReply: true,
    },
    {
      id: 5,
      text: "It'll push your limits. The AV's faster than any human opponent. Perfect for honing those reflexes.",
      isReply: false,
    },
    {
      id: 6,
      text: "Sounds intense. I'll make sure I'm well-rested before the session.",
      isReply: true,
    },
    {
      id: 7,
      text: "Good thinking. Oh, and don't forget to calibrate your optical implants. You'll need razor-sharp vision for this.",
      isReply: false,
    },
  ],
  "3-3": [
    {
      id: 1,
      text: "Got a line on some new chrome. Military-grade reflex enhancers. Interested?",
      isReply: false,
    },
    { id: 2, text: "Sounds preem. What's the catch?", isReply: true },
    {
      id: 3,
      text: "It's not exactly... approved for civilian use. But it'll give you an edge in the ring.",
      isReply: false,
    },
    {
      id: 4,
      text: "Risky, but I'm in. When can we do the installation?",
      isReply: true,
    },
    {
      id: 5,
      text: "I'll set it up for next week. And V? Keep this on the down-low. If the league finds out, we're both in deep shit.",
      isReply: false,
    },
    {
      id: 6,
      text: "No worries, Coach. I know how to keep a secret. Any side effects I should be aware of?",
      isReply: true,
    },
    {
      id: 7,
      text: "Might be some insomnia, increased aggression. Nothing you can't handle. Just... try not to punch any civilians, alright?",
      isReply: false,
    },
  ],
  "4-1": [
    {
      id: 1,
      text: "Alright, kid. Big score coming up. Arasaka tower. We need to grab a prototype biochip.",
      isReply: false,
    },
    {
      id: 2,
      text: "Arasaka? That's major leagues, Dex. What's our angle?",
      isReply: true,
    },
    {
      id: 3,
      text: "Got an inside woman. Evelyn Parker. She'll provide the intel. Meet her at the Lizzie's Bar tomorrow, 2200.",
      isReply: false,
    },
    {
      id: 4,
      text: "I'm on it. This could be our ticket to the big time.",
      isReply: true,
    },
    {
      id: 5,
      text: "Just remember, V. In the big leagues, the bigger you are, the harder you fall. Stay frosty.",
      isReply: false,
    },
    {
      id: 6,
      text: "Always do, Dex. Any other details I should know before meeting Evelyn?",
      isReply: true,
    },
    {
      id: 7,
      text: "Yeah, she's a doll at Clouds. Sharp as a tack, but watch your back. Dolls are walking databases of secrets.",
      isReply: false,
    },
  ],
  "4-2": [
    {
      id: 1,
      text: "Let's talk eddies. This job's gonna make you rich, kid.",
      isReply: false,
    },
    { id: 2, text: "I'm listening. What's the split?", isReply: true },
    {
      id: 3,
      text: "60-40. My favor. But 40% of this score is more than most mercs see in a year.",
      isReply: false,
    },
    {
      id: 4,
      text: "Fair enough. As long as it all goes smoothly.",
      isReply: true,
    },
    {
      id: 5,
      text: "That's the spirit. Do this right, and you'll never have to work another day in your life.",
      isReply: false,
    },
    {
      id: 6,
      text: "Sounds good, Dex. Any idea on the total take?",
      isReply: true,
    },
    {
      id: 7,
      text: "Let's just say... seven figures. Minimum. But keep that to yourself, got it?",
      isReply: false,
    },
  ],
  "4-3": [
    {
      id: 1,
      text: "V, we need to talk escape routes. Once we have the biochip, Arasaka's gonna be all over us.",
      isReply: false,
    },
    {
      id: 2,
      text: "I thought we were going in quiet. No alarms, no pursuit.",
      isReply: true,
    },
    {
      id: 3,
      text: "Plan for the worst, hope for the best. I've got a fixer setting up a series of safehouses across Night City.",
      isReply: false,
    },
    {
      id: 4,
      text: "Smart. Any specific route you want me to memorize?",
      isReply: true,
    },
    {
      id: 5,
      text: "Sending you the details now. Memorize them, then delete the file. Can't risk this falling into the wrong hands.",
      isReply: false,
    },
    {
      id: 6,
      text: "Got it. I'll study it tonight. Anything else I should prep?",
      isReply: true,
    },
    {
      id: 7,
      text: "Yeah, get yourself some high-grade stims. If things go south, you might need to stay awake for a long time.",
      isReply: false,
    },
  ],
  "5-1": [
    {
      id: 1,
      text: "V, this Arasaka job... I've got a bad feeling, hermano.",
      isReply: false,
    },
    {
      id: 2,
      text: "It's a big risk, Jackie, but think of the payout. We could finally be set for life.",
      isReply: true,
    },
    {
      id: 3,
      text: "I know, I know. Just... promise me you'll be careful. We're in over our heads here.",
      isReply: false,
    },
    {
      id: 4,
      text: "We've got this, Jackie. In and out, clean and quiet. Trust me.",
      isReply: true,
    },
    {
      id: 5,
      text: "I do trust you, V. It's everyone else I'm worried about. Especially Dex. Guy's got a reputation.",
      isReply: false,
    },
    {
      id: 6,
      text: "I hear you. We'll watch each other's backs, like always. It'll be fine.",
      isReply: true,
    },
    {
      id: 7,
      text: "Yeah, you're right. Just... let's make it our last job, okay? I'm thinking of settling down with Misty.",
      isReply: false,
    },
  ],
  "5-2": [
    {
      id: 1,
      text: "Hey V, mama's asking if you're coming to the Coyote this weekend. Says she misses your ugly mug.",
      isReply: false,
    },
    {
      id: 2,
      text: "Wouldn't miss it for the world. Your mom's tamales are the best in Night City.",
      isReply: true,
    },
    {
      id: 3,
      text: "Haha, I'll let her know. She'll probably try to fatten you up again.",
      isReply: false,
    },
    {
      id: 4,
      text: "Fine by me. It's been too long since I had a proper meal. See you there, Jackie.",
      isReply: true,
    },
    {
      id: 5,
      text: "Preem! Oh, and heads up - Misty might be there. Try not to put your foot in your mouth this time, eh?",
      isReply: false,
    },
    {
      id: 6,
      text: "Hey, that was one time! I'll be on my best behavior, promise.",
      isReply: true,
    },
    {
      id: 7,
      text: "You better be. Oh, and bring some of that tequila we... found. You know the one.",
      isReply: false,
    },
  ],
  "5-3": [
    {
      id: 1,
      text: "V, need a favor. My sister's in trouble with some Valentinos. Can you help?",
      isReply: false,
    },
    { id: 2, text: "Of course, Jackie. What's the situation?", isReply: true },
    {
      id: 3,
      text: "She borrowed some eddies, couldn't pay back in time. They're threatening to repo her car.",
      isReply: false,
    },
    {
      id: 4,
      text: "Got it. I'll talk to them, see if we can work something out. Your family's my family, Jackie.",
      isReply: true,
    },
    {
      id: 5,
      text: "Thanks, V. I owe you one. Just... try to handle it peacefully, yeah? Don't want this escalating into a gang war.",
      isReply: false,
    },
    {
      id: 6,
      text: "No promises, but I'll do my best. Where can I find these Valentinos?",
      isReply: true,
    },
    {
      id: 7,
      text: "They hang out at the El Coyote Cojo. Ask for Gustavo. And V? Thanks again, choom.",
      isReply: false,
    },
  ],
  "5-4": [
    {
      id: 1,
      text: "V! You gotta see this beauty I just scored. A Quadra Type-66. She purrs like a kitten.",
      isReply: false,
    },
    {
      id: 2,
      text: "No way! How'd you manage that? Those are rare as hell.",
      isReply: true,
    },
    {
      id: 3,
      text: "Let's just say I know a guy who knows a guy. Want to take her for a spin?",
      isReply: false,
    },
    {
      id: 4,
      text: "Hell yeah! Meet you at the Megabuilding in 20?",
      isReply: true,
    },
    {
      id: 5,
      text: "You got it, choom. Prepare to have your mind blown. This ride's gonna change our lives.",
      isReply: false,
    },
    {
      id: 6,
      text: "Can't wait. Hey, think we could use it for the Arasaka job?",
      isReply: true,
    },
    {
      id: 7,
      text: "Are you loco? This baby's too recognizable. But after the job? We'll cruise the city in style, amigo.",
      isReply: false,
    },
  ],
});

const currentContactId = ref(2);
const currentTopicId = ref(1);
const currentMessages = computed(
  () =>
    messages.value[`${currentContactId.value}-${currentTopicId.value}`] || []
);
const messageContainer = ref(null);

const scrollToBottom = () => {
  nextTick(() => {
    if (messageContainer.value) {
      messageContainer.value.scrollTop = messageContainer.value.scrollHeight;
    }
  });
};

const toggleContact = (contactId) => {
  const contact = contacts.value.find((c) => c.id === contactId);
  if (contact) {
    contact.expanded = !contact.expanded;
  }
};

const selectTopic = (contactId, topicId) => {
  contacts.value.forEach((contact) => {
    contact.topics.forEach((topic) => {
      topic.active = contact.id === contactId && topic.id === topicId;
    });
  });
  currentContactId.value = contactId;
  currentTopicId.value = topicId;
  nextTick(() => {
    scrollToBottom();
  });
};

const newMessage = ref("");

const sendMessage = () => {
  if (newMessage.value.trim()) {
    const newId = currentMessages.value.length + 1;
    const messageKey = `${currentContactId.value}-${currentTopicId.value}`;
    if (!messages.value[messageKey]) {
      messages.value[messageKey] = [];
    }
    messages.value[messageKey].push({
      id: newId,
      text: newMessage.value,
      isReply: true,
    });
    newMessage.value = "";
    scrollToBottom();
  }
};

const formatMessage = (text) => {
  const urlRegex = /(https?:\/\/[^\s]+)/g;
  return text.replace(
    urlRegex,
    (url) =>
      `<a href="${url}" target="_blank" class="text-blue-400 hover:underline">${url}</a>`
  );
};

onMounted(() => {
  nextTick(() => {
    scrollToBottom();
  });
});
</script>

<style>
@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.animate-fadeIn {
  animation: fadeIn 0.3s ease-out;
}

/* Custom scrollbar */
::-webkit-scrollbar {
  width: 8px;
}

::-webkit-scrollbar-track {
  background: rgba(139, 0, 0, 0.1);
}

::-webkit-scrollbar-thumb {
  background: rgba(0, 229, 255, 0.3);
  border-radius: 4px;
}

::-webkit-scrollbar-thumb:hover {
  background: rgba(0, 229, 255, 0.5);
}

.slide-enter-active,
.slide-leave-active {
  transition: max-height 0.3s ease, opacity 0.3s ease;
  overflow: hidden;
}

.slide-enter-from,
.slide-leave-to {
  max-height: 0;
  opacity: 0;
}

.slide-enter-to,
.slide-leave-from {
  max-height: 1000px;
  opacity: 1;
}
</style>
