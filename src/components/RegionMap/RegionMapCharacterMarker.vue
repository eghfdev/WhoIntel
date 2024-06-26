<template>
	<svg>
		<g v-for="marker in characterMarkers" :key="marker.name">
			<ellipse
				:cx="marker.cx" :cy="marker.cy"
				:rx="marker.rx" :ry="marker.ry"
				:fill="marker.color"
			/>
		</g>
	</svg>
</template>

<script lang="ts">
import {Component, Vue} from "vue-property-decorator"
import systemManager from "@/service/SystemManager"
import characterManager from "@/service/CharacterManager"
import Character from "@/lib/Character"

@Component
export default class RegionMapCharacterMarker extends Vue {
	get characterMarkers(): any[] {
		if (!systemManager.currentRegion) return []
		const systemsToCharacters = characterManager.regionSystemToChars[systemManager.currentRegion.id]

		// to highlight neighbour regions too
		if (characterManager.activeCharacter && characterManager.activeCharacter.location) {
			characterManager.activeCharacter.location.neighbourRegions.forEach(neighbourRegionId => {
				const systemsToCharactersNeighbour = characterManager.regionSystemToChars[neighbourRegionId]
				if (systemsToCharactersNeighbour) {
					for (const [systemID, characters] of Object.entries(systemsToCharactersNeighbour)) {
						if (characters) {
							systemsToCharacters[systemID] = characters
						}
					}
				}
			})
		}

		if (!systemsToCharacters) return []

		const result: any[] = []

		for (const [, characters] of Object.entries(systemsToCharacters)) {
			characters.forEach(character => {
				const system = (character as Character).location

				if (!system || !system.mapCoordinates) {
					return
				}

				result.push({
					cx: system.mapCoordinates.center_x - 2.5,
					cy: system.mapCoordinates.center_y,
					rx: (system.mapCoordinates.width / 2) + 4,
					ry: (system.mapCoordinates.height / 2) + 4,
					color: character.name === characterManager.activeCharacter?.name ? "#FB00FF" : "#8B008D",
					name: character.name
				})
			})
		}

		return result
	}
}
</script>
