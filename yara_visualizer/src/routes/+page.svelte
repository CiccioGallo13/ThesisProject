<script lang="ts">
    import { Button, Card, CardBody, CardFooter, CardHeader, CardTitle, Col, Container, FormGroup, FormText, Input, Label, Row, Styles, Table } from 'sveltestrap';
    import type { Color } from 'sveltestrap/src/shared';
    import { _sendData } from './+page';
    import type { JsonRequest, JsonResponse, Match, StringMatch, StringMatchIntance } from '../model/model';

    const color: Color = 'dark';
    let files: FileList;
    let dataTextArea: string;
    let rulesTextArea: string;
    let completeScan: boolean = false;
    let renderTable: boolean = false;
    function fileScan(who: string): any{
        var result: string  = '';
        if(files){
            
            for(const file of files){
                
                let reader= new FileReader();
                reader.readAsText(file);
                
                reader.onload = function(){
                    //console.log(reader.result); 
                    result = reader.result?.slice(0) as string;
                    if(who === "data"){
                        console.log(result);
                        dataTextArea= result;
                    }else{
                        if(who === "rules")
                        {
                            rulesTextArea = result;
                        }
                    }
                };
                
                reader.onerror = function() {
                    console.log(reader.error);
                };
            }
            
        }
    }
    let matches: JsonResponse;
    
    async function scanData() {
        let jsonRequest: JsonRequest = {
            rules: rulesTextArea,
            data: dataTextArea,
            complete_scan: completeScan
        };

        console.log(jsonRequest);
        
        _sendData(jsonRequest).then((response) => {
            matches = response;
            console.log(response);
            renderTable = true;
        
        }
        , (error) => {
        console.log(error);
        });

    }

</script>

<Styles />

<Card class="mx-auto" style="border-radius: 0;">
    <CardHeader>
        <CardTitle style="text-align: center;">YARA VISUALIZER</CardTitle>
    </CardHeader>
    <CardBody>
        <Row cols={2}>
            <Container>
                <Col sm={{ size: 'auto', offset: 1 }}>
                    <FormGroup>
                        <Label for="rulesTextArea">Rules</Label>
                        <Input type="textarea" name="text" id="rulesTextArea" bind:value={rulesTextArea} style="min-height:100pt; max-height:400pt"/>
                    </FormGroup>
                    <div class="options">
                        <FormGroup>
                            <Label for="rulesFile">or upload your rules here</Label>
                            <Input bind:files type="file" name="file" id="rulesFile" on:change={() => fileScan("rules")}/>
                            <FormText color="muted" />
                        </FormGroup>

                    </div>

                </Col>
            </Container>

            <Container>
                <Col sm={{ size: 'auto', offset: 1}}>
                    <FormGroup>
                        <Label for="dataTextArea">Data</Label>
                        <Input type="textarea" name="text" id="dataTextArea" bind:value={dataTextArea} style="min-height:100pt; max-height:400pt"/>
                    </FormGroup>
                    <div class="options">
                        <FormGroup>
                            <Label for="dataFile">or upload your file to scan here</Label>
                            <Input bind:files type="file" name="file" id="dataFile" on:input={() => fileScan("data")} />
                            <FormText color="muted" />
                        </FormGroup>
                    </div>
                </Col>
            </Container>

        </Row>
    </CardBody>
    
    <CardFooter>
    <Row cols={1}>
    <div class="centered" style="padding-top: 0cm;">
        <Col sm={{ size: 'auto'}}>
            <FormGroup>
                <Input id="completeScanCheckBox" type="checkbox" bind:checked={completeScan} label="Complete scan" />
            </FormGroup>
        </Col>
    </div>
    
    <div class="centered">
        <Col sm={{ size: 'auto'}}>
            <Button {color} style="width: 130pt" on:click={scanData}>Scan</Button>
        </Col>
    </div>
    </Row>
</CardFooter>

</Card>
{#if renderTable}
<Container>
<div class="options">
<Table>
    <thead>
        <tr>
            <th>Encoding</th>
            <th>Data</th>
            <th>Matched rules</th>
        </tr>
    </thead>
    <tbody>
        {#each matches.encoding_matches as match}
        <tr>
            <td>{match.encoding}</td>
            <td>
                {#each match.matches as dataMatch}
                    {#each dataMatch.string_match as stringMatch}
                        {#each stringMatch.instances as instance}
                            <p>{instance.matched_data}</p>
                        {/each}
                    {/each}
                {/each}
            </td>
            <td>
                {#each match.matches as dataMatch}
                    <p>{dataMatch.rule}</p>
                {/each}
            </td>
        </tr>
        {/each}
    </tbody>
</Table>
</div>
</Container>
{/if}



<style>
    .options {
        padding-top: 0.6cm;
    }
    .centered {
        display: flex;
        justify-content: center;
        align-items: center;
    }
</style>