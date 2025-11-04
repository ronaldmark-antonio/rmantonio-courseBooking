<script setup>

	import { onBeforeMount } from 'vue';

	import { useGlobalStore } from '../stores/global'; // <<
	import { useRouter } from 'vue-router'; // <<
	const {getUserDetails} = useGlobalStore(); // << 

	const router = useRouter()
	
	//This will run before the component is displayed
	onBeforeMount(()=>{

		//clears the localStorage
		localStorage.clear();
		getUserDetails(null); // << 

		//Allows us to redirect the user.
		// window.location.href = "/login"

		router.push({path: "/login"});

	})

</script>